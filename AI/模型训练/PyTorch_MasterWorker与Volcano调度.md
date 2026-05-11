[[模型训练]]
# PyTorch Master/Worker 分布式训练 & Volcano 调度实践

> **调研日期**：2026-04-28  
> **关键词**：PyTorch DDP, torchrun, Master/Worker, Volcano Scheduler, Gang Scheduling, Kubernetes  
> **适用场景**：多节点多 GPU 分布式训练在 K8s 上的生产部署

---

## 1. 概述

在 Kubernetes 上运行 PyTorch 分布式训练时，需要解决两个核心问题：

1. **PyTorch 层**：如何编排 Master 和 Worker 角色的分布式训练进程
2. **K8s 调度层**：如何确保所有 Pod 能同时被调度、资源不浪费

**Volcano**（CNCF 孵化项目）通过 **Gang Scheduling** 和 **Job 级调度**完美解决了 Kubernetes 原生调度器对 AI 训练场景的不足。

---

## 2. PyTorch Master/Worker 工作方式

### 2.1 核心概念

PyTorch 分布式训练中，所有节点角色为 **Worker**（无严格 Master/Worker 区分），但需要约定一个协调者：

| 概念 | 说明 |
|------|------|
| **MASTER_ADDR** | rank 0 的节点地址，用于进程初始化握手 |
| **MASTER_PORT** | rank 0 上监听的端口 |
| **WORLD_SIZE** | 所有进程中参与训练的进程总数（nnodes × nproc_per_node） |
| **RANK** | 进程全局唯一编号（0 ~ WORLD_SIZE-1） |
| **LOCAL_RANK** | 进程在本地节点上的编号（0 ~ nproc_per_node-1） |
| **NODE_RANK** | 节点在集群中的编号 |
| **nnodes** | 参与训练的节点总数 |
| **nproc_per_node** | 每个节点上运行的进程数（通常 = GPU 数） |

### 2.2 `torchrun`：推荐的启动方式

从 PyTorch 1.9+ 起，官方推荐用 `torchrun`（替代旧的 `torch.distributed.launch`）来启动分布式训练。

#### 关键参数

```bash
torchrun \
    --nnodes=4                    # 节点总数
    --nproc_per_node=8            # 每节点进程数（通常=GPU数）
    --node_rank=0                 # 当前节点编号（仅master需要显式设置）
    --rdzv_backend=c10d           # 推荐 rendezvous 后端
    --rdzv_endpoint=master-addr:29500  # 协调服务端点
    --rdzv_id=job-unique-id       # 作业唯一标识
    --max_restarts=3              # 最大重启次数（容错）
    train.py
```

#### 环境变量注入

`torchrun` 自动为每个进程注入以下环境变量：

- `RANK` — 全局进程编号
- `LOCAL_RANK` — 本地进程编号（用于指定 GPU）
- `WORLD_SIZE` — 总进程数
- `MASTER_ADDR` — rank 0 的主机地址
- `MASTER_PORT` — rank 0 的监听端口
- `LOCAL_WORLD_SIZE` — 本地进程数
- `GROUP_RANK` — WorkerGroup 编号（多节点场景）

#### 训练代码示例

```python
import os
import torch
import torch.distributed as dist

def main():
    # torchrun 自动注入环境变量，直接初始化即可
    dist.init_process_group(backend="nccl")

    local_rank = int(os.environ["LOCAL_RANK"])
    global_rank = int(os.environ["RANK"])
    world_size = int(os.environ["WORLD_SIZE"])

    torch.cuda.set_device(local_rank)
    device = torch.device("cuda", local_rank)

    model = MyModel().to(device)
    ddp_model = torch.nn.parallel.DistributedDataParallel(
        model, device_ids=[local_rank], output_device=local_rank
    )

    # 训练逻辑...
    # 注意：所有进程运行同一份代码，dist 后端自动处理同步

    dist.destroy_process_group()

if __name__ == "__main__":
    main()
```

### 2.3 传统手动方式（不推荐）

在没有 `torchrun` 时，手动使用 `torch.distributed.launch`：

```bash
python -m torch.distributed.launch \
    --nnodes=4 \
    --nproc_per_node=8 \
    --node_rank=$RANK \
    --master_addr="$MASTER_ADDR" \
    --master_port=29500 \
    train.py
```

但这种方式的 `node_rank` 必须在启动时手动设定，不适合自动编排。

### 2.4 容错与弹性训练（TorchElastic）

`torchrun` 基于 **TorchElastic** 框架，支持：

- **固定大小容错**：`--nnodes=4` + `--max-restarts=N`，最多容忍 N 次失败重启
- **弹性训练**：`--nnodes=1:4` 表示最少 1 个节点、最多 4 个节点，节点数动态变化

---

## 3. Volcano 调度器

### 3.1 解决了什么？

| 问题 | 说明 | Volcano 解决方案 |
|------|------|------------------|
| **Gang Scheduling 缺失** | K8s 默认调度器逐个调度 Pod，部分调度导致 GPU 闲置 | `minAvailable` 确保全部就绪后才分配资源 |
| **死锁问题** | 多个分布式任务互相抢占但都不够启动 | Gang + Queue 策略彻底避免 |
| **公平调度** | 多团队共享集群时缺少配额管理 | Queue + weight + DRF 插件 |
| **抢占/优先级** | 低优先级任务可能阻塞高优训练 | PriorityClass + 抢占策略 |

### 3.2 核心架构

```
┌─────────────────────────────────────────────────┐
│              Volcano 整体架构                    │
├─────────────┬───────────────┬───────────────────┤
│  Scheduler  │   Controller  │ Admission Webhook │
│  (调度决策)  │   (CRD管理)    │  (校验/注入默认值)  │
│  Gang/DRF/  │   Job/Queue/  │                   │
│  Binpack    │   PodGroup    │                   │
└─────────────┴───────────────┴───────────────────┘
```

### 3.3 关键 CRD

#### Queue（队列）

```yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: ml-research-queue
spec:
  weight: 4                  # 资源分配权重
  reclaimable: true          # 闲置资源可被其他队列借用
  capability:
    nvidia.com/gpu: 32
    cpu: "128"
    memory: "512Gi"
```

#### PodGroup（Pod 组 — 自动生成）

定义了一组紧密耦合的 Pod，是 Gang Scheduling 的基本单位。VolcanoJob 创建时自动生成。

```yaml
spec:
  minMember: 4              # 最少 Pod 数
  minResources:             # 最少资源要求
    nvidia.com/gpu: 4
  queue: ml-research-queue
```

### 3.4 Volcano 调度插件

| 插件 | 功能 | 适用场景 |
|------|------|----------|
| **Gang** | All-or-Nothing 调度 | 分布式训练、MPI |
| **DRF** | 基于主导资源的公平调度 | 多租户 |
| **Proportion** | 队列间按权重分配 | 团队资源隔离 |
| **Priority** | 按优先级调度 | 生产/开发混合 |
| **Binpack** | 最大化节点利用率 | 资源优化 |
| **Predicates** | GPU 资源预过滤 | GPU 密集型任务 |
| **NUMA-aware** | NUMA 拓扑感知 | HPC 场景 |

---

## 4. Volcano + PyTorch 集成方案

### 4.1 关键插件

在 VolcanoJob 中需要启用的插件：

| 插件 | 作用 |
|------|------|
| `env: []` | 注入 `VK_TASK_INDEX` / `VC_TASK_INDEX` 环境变量（用于识别 Pod 在 Task 中的序号） |
| `svc: []` | 创建 Headless Service，使 Pod 间可以通过 `{task-name}-{index}.{job-name}` 域名互相发现 |
| `ssh: []` | 在 Pod 间配置 SSH（MPI 场景需要，PyTorch 一般不需要） |

### 4.2 自动注入的环境变量

**svc plugin** 自动为 Pod 添加的发现变量：

| 环境变量 | 说明 |
|----------|------|
| `VC_MASTER_HOST` | Master Task 的第一个 Pod 的域名 |
| `{TASK_NAME}_HOST` | 各 Task 的 host 文件路径（`/etc/volcano/{task}.host`） |

**env plugin** 自动添加的索引变量：

| 环境变量 | 说明 |
|----------|------|
| `VK_TASK_INDEX` | Pod 在 Task 中的序号（0-based） |
| `VC_TASK_INDEX` | 同上（VK_ 后续会废弃） |

**svc plugin** 创建的域名规则：

```
# Pod 域名格式
{task-name}-{index}.{job-name}.{namespace}.svc.cluster.local

# 例如 master 的域名
master-0.pytorch-ddp-training.default.svc.cluster.local
```

### 4.3 两种集成模式

#### 模式 A：传统 Master/Worker 分角色（带 node_rank）

适用于需要显式区分 Master 和 Worker 角色的场景，通过 `torchrun --node_rank` 区分。

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: pytorch-ddp-training
spec:
  minAvailable: 5                  # 必须 5 个 Pod 全部就绪才调度
  schedulerName: volcano
  queue: ml-research-queue
  plugins:
    env: []                        # 注入 VK_TASK_INDEX
    svc: []                        # 创建 Headless Service
  policies:
    - event: PodEvicted
      action: RestartJob
  tasks:
    - replicas: 1
      name: master
      template:
        spec:
          containers:
            - name: pytorch
              image: pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime
              command:
                - torchrun
              args:
                - "--nnodes=5"
                - "--nproc_per_node=1"
                - "--node_rank=0"      # Master 显式设为 0
                - "train.py"
              resources:
                limits:
                  nvidia.com/gpu: 1
              env:
                - name: MASTER_ADDR
                  value: "master-0.pytorch-ddp-training.default.svc.cluster.local"
                - name: MASTER_PORT
                  value: "29500"
          restartPolicy: OnFailure

    - replicas: 4
      name: worker
      template:
        spec:
          containers:
            - name: pytorch
              image: pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime
              command:
                - torchrun
              args:
                - "--nnodes=5"
                - "--nproc_per_node=1"
                - "--node_rank=$(VK_TASK_INDEX)"   # Worker 的 node_rank = 1,2,3,4
                - "train.py"
              resources:
                limits:
                  nvidia.com/gpu: 1
              env:
                - name: MASTER_ADDR
                  value: "master-0.pytorch-ddp-training.default.svc.cluster.local"
                - name: MASTER_PORT
                  value: "29500"
          restartPolicy: OnFailure
```

**缺点**：需要硬编码所有 `--nnodes`。每次增减节点都要修改 YAML。

***带 plugin 版本:***
```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: pytorch-job
spec:
  minAvailable: 3 # 最小可用副本数
  schedulerName: volcano # 使用Volcano调度器
  plugins:
    pytorch: ["--master=master", "--worker=worker", "--port=23456"] # 启用PyTorch Plugin
  queue: default # 指定队列
  tasks:
    - replicas: 1
      name: master # Master任务定义
      policies:
        - event: TaskCompleted
          action: CompleteJob # 当Master任务完成时，整个Job标记为完成
      template:
        spec:
          containers:
            - name: master
              image: idocker.io/pytorch/pytorch:2.0.0-cuda11.7-cudnn8-runtime
              imagePullPolicy: IfNotPresent
              command:
                - python
                - /workspace/pytorch-demo.py
              env:
                - name: PYTHONUNBUFFERED
                  value: "1"
              volumeMounts:
                - name: training-script
                  mountPath: /workspace
              resources:
                requests:
                  cpu: "1"
                  memory: "2Gi"
                limits:
                  cpu: "1"
                  memory: "2Gi"
          volumes:
            - name: training-script
              configMap:
                name: pytorch-demo
          restartPolicy: OnFailure
    - replicas: 2
      name: worker # Worker任务定义
      template:
        spec:
          containers:
            - name: worker
              image: idocker.io/pytorch/pytorch:2.0.0-cuda11.7-cudnn8-runtime
              imagePullPolicy: IfNotPresent
              command:
                - python
                - /workspace/pytorch-demo.py
              env:
                - name: PYTHONUNBUFFERED
                  value: "1"
              volumeMounts:
                - name: training-script
                  mountPath: /workspace
              resources:
                requests:
                  cpu: "1"
                  memory: "2Gi"
                limits:
                  cpu: "1"
                  memory: "2Gi"
          volumes:
            - name: training-script
              configMap:
                name: pytorch-demo
          restartPolicy: OnFailure

```
#### 模式 B：统一角色 + c10d Rendezvous（推荐）

所有节点运行相同的 `torchrun` 命令，使用 c10d 的 rendezvous 机制自动发现成员。

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: pytorch-distributed-training
spec:
  minAvailable: 4                     # Gang: 4 个 Pod 全部就绪
  schedulerName: volcano
  queue: training-queue
  maxRetry: 3
  ttlSecondsAfterFinished: 3600
  plugins:
    env: []
    svc: []                           # 提供 Pod 间域名发现
  policies:
    - event: PodEvicted
      action: RestartJob
    - event: PodFailed
      action: RestartJob
    - event: TaskCompleted
      action: CompleteJob
  tasks:
    - name: worker                    # 统一角色名，所有节点一视同仁
      replicas: 4                     # 4 节点 × N GPU
      template:
        spec:
          containers:
            - name: trainer
              image: registry.example.com/trainer:latest
              command:
                - torchrun
              args:
                - "--nnodes=4"
                - "--nproc_per_node=4"    # 每节点 4 GPU
                - "--rdzv_backend=c10d"
                - "--rdzv_endpoint=$(VC_MASTER_HOST):29500"
                - "--max_restarts=3"
                - "train.py"
              resources:
                limits:
                  nvidia.com/gpu: 4
              volumeMounts:
                - name: dshm
                  mountPath: /dev/shm
          volumes:
            - name: dshm
              emptyDir:
                medium: Memory
                sizeLimit: 64Gi
          restartPolicy: OnFailure
```

> **关键点**：`$(VC_MASTER_HOST)` 是 Volcano svc plugin 注入的环境变量，指向该 Job 中第 0 个 Pod 的域名。c10d rendezvous 在该地址上建立协调服务，所有节点自动入组。

#### 模式 C：弹性训练（Elastic）

支持节点数动态变化的场景，降低资源碎片。

```yaml
spec:
  tasks:
    - name: worker
      replicas: 6                     # 最多 6 个节点
      template:
        spec:
          containers:
            - name: trainer
              command:
                - torchrun
              args:
                - "--nnodes=3:6"            # 最少 3，最多 6
                - "--nproc_per_node=4"
                - "--rdzv_backend=c10d"
                - "--rdzv_endpoint=$(VC_MASTER_HOST):29500"
                - "--max_restarts=5"
                - "--rdzv_min_workers=12"    # 最少 12 个 worker 才启动
                - "train.py"
```

---

## 5. 完整示例项目

### 5.1 训练代码

```python
# train.py
import os
import argparse
import torch
import torch.distributed as dist
import torch.nn as nn
import torch.optim as optim
from torch.nn.parallel import DistributedDataParallel as DDP

class SimpleModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(784, 256),
            nn.ReLU(),
            nn.Linear(256, 10),
        )

    def forward(self, x):
        return self.net(x)

def cleanup():
    dist.destroy_process_group()

def setup():
    # torchrun 自动注入了环境变量：
    #   RANK, LOCAL_RANK, WORLD_SIZE, MASTER_ADDR, MASTER_PORT
    dist.init_process_group(backend="nccl")

def main():
    setup()
    local_rank = int(os.environ["LOCAL_RANK"])
    global_rank = int(os.environ["RANK"])
    world_size = int(os.environ["WORLD_SIZE"])

    torch.cuda.set_device(local_rank)
    device = torch.device("cuda", local_rank)

    model = SimpleModel().to(device)
    ddp_model = DDP(model, device_ids=[local_rank])

    optimizer = optim.SGD(ddp_model.parameters(), lr=0.01)
    loss_fn = nn.CrossEntropyLoss()

    # 每个 epoch 确保所有进程同步
    for epoch in range(10):
        # 模拟训练
        dummy_input = torch.randn(64, 784).to(device)
        dummy_target = torch.randint(0, 10, (64,)).to(device)

        optimizer.zero_grad()
        output = ddp_model(dummy_input)
        loss = loss_fn(output, dummy_target)
        loss.backward()
        optimizer.step()

        if global_rank == 0:
            print(f"Epoch {epoch}, Loss: {loss.item():.4f}")

        # 分布式训练中通常需要 barrier 同步
        dist.barrier()

    cleanup()

if __name__ == "__main__":
    main()
```

### 5.2 Volcano Queue

```yaml
# 01-queue.yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: training-queue
spec:
  weight: 4
  reclaimable: true
  capability:
    nvidia.com/gpu: 16
    cpu: "64"
    memory: "256Gi"
```

### 5.3 Dockerfile

```dockerfile
FROM pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime

WORKDIR /workspace
COPY train.py .

ENTRYPOINT ["torchrun"]
```

### 5.4 VolcanoJob（完整版）
```yaml
# 02-volcano-job.yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: pytorch-distributed-training
  namespace: default
spec:
  minAvailable: 4
  schedulerName: volcano
  queue: training-queue
  maxRetry: 3
  ttlSecondsAfterFinished: 3600
  plugins:
	pytorch: ["--master=master", "--worker=worker", "--port=23456"] # 启用PyTorch Plugin
    env: []
    svc: []
  policies:
    - event: PodEvicted
      action: RestartJob
    - event: PodFailed
      action: RestartJob
    - event: TaskCompleted
      action: CompleteJob
  tasks:
    - name: worker
      replicas: 4
      template:
        spec:
          containers:
            - name: trainer
              image: myregistry/torch-trainer:latest
              command:
                - torchrun
              args:
                - "--nnodes=4"
                - "--nproc_per_node=4"
                - "--rdzv_backend=c10d"
                - "--rdzv_endpoint=$(VC_MASTER_HOST):29500"
                - "--max_restarts=3"
                - "train.py"
              resources:
                limits:
                  nvidia.com/gpu: 4
                  cpu: 16
                  memory: 128Gi
                requests:
                  nvidia.com/gpu: 4
                  cpu: 16
                  memory: 128Gi
              env:
                - name: NCCL_DEBUG
                  value: "INFO"
                - name: NCCL_SOCKET_IFNAME
                  value: "eth0"
                - name: GLOO_SOCKET_IFNAME
                  value: "eth0"
              volumeMounts:
                - name: dshm
                  mountPath: /dev/shm
                - name: training-data
                  mountPath: /data
                - name: checkpoints
                  mountPath: /checkpoints
          volumes:
            - name: dshm
              emptyDir:
                medium: Memory
                sizeLimit: 64Gi
            - name: training-data
              persistentVolumeClaim:
                claimName: training-data-pvc
            - name: checkpoints
              persistentVolumeClaim:
                claimName: checkpoint-pvc
          restartPolicy: OnFailure
```

### 5.5 部署命令

```bash
# 1. 安装 Volcano（如果未安装）
helm repo add volcano-sh https://volcano-sh.github.io/helm-charts
helm repo update
helm install volcano volcano-sh/volcano -n volcano-system --create-namespace

# 2. 创建 Queue
kubectl apply -f 01-queue.yaml

# 3. 创建训练数据 PVC
kubectl apply -f pvc.yaml

# 4. 提交训练任务
kubectl apply -f 02-volcano-job.yaml

# 5. 查看任务状态
kubectl get vcjob pytorch-distributed-training -o yaml

# 6. 查看 Pod 状态
kubectl get pods -l "volcano.sh/job-name=pytorch-distributed-training"

# 7. 查看日志
kubectl logs -f pytorch-distributed-training-worker-0

# 8. 查看队列状态
kubectl get queue training-queue -o yaml | grep -A5 status
```

### 5.6 训练全过程运行流

```
1. 用户 kubectl apply -f 02-volcano-job.yaml
                           │
2. Volcano Admission Webhook 校验并注入默认值
                           │
3. Volcano Controller 创建 PodGroup + Pod 资源
   ┌─ PodGroup: pytorch-distributed-training-pg
   │   minMember: 4
   │   queue: training-queue
   └─ Pod × 4 (worker-0 ~ worker-3)
                           │
4. Volcano Scheduler 执行 Gang Scheduling
   ├─ 检查是否有足够资源运行全部 4 个 Pod
   ├─ 资源不足 → 全部挂起，等待资源释放
   ├─ 资源充足 → 4 个 Pod 同时绑定到 Node
   └─ 同时创建 Headless Service（svc plugin）
                           │
5. Pods 启动运行 torchrun
   ├─ Pod worker-3: torchrun --nnodes=4 --nproc_per_node=4
   │   └─ 内部 spawn 4 个进程 (local_rank 0~3)
   │       └─ train.py → dist.init_process_group("nccl")
   ├─ Pod worker-2: (同上)
   ├─ Pod worker-1: (同上)
   └─ Pod worker-0: (同上，同时作为 rendezvous host)
                           │
6. c10d Rendezvous 协调
   ├─ worker-0: VC_MASTER_HOST → worker-0 域名
   │   在 29500 端口建立 TCP Store
   ├─ worker-1/2/3: 通过 $VC_MASTER_HOST:29500 加入
   └─ 全部 16 进程形成 WORLD_SIZE=16 的 WorkerGroup
                           │
7. 分布式训练开始
   ├─ 每个 epoch: forward → backward → allreduce → optimizer.step
   ├─ NCCL allreduce 同步梯度（16 GPU 并行）
   └─ rank 0 定期保存 checkpoint
                           │
8. 训练完成 → Pod 进入 Completed
   └─ ttlSecondsAfterFinished 后自动清理
```

---

## 6. Volcanic 自动注入环境变量详解

### svc plugin 注入变量

```
# 自动写入每个 Pod 的 /etc/volcano/ 目录
/etc/volcano/worker.host    # 内容：所有 worker Pod 的域名，每行一个
/etc/volcano/master.host    # 如果有 master task 则生成

# 自动注入环境变量
VC_MASTER_HOST=worker-0.pytorch-distributed-training.default.svc.cluster.local
# 指向第 0 个 Pod（按字典序），用于 rendezvous 协调
```

### env plugin 注入变量

```
VK_TASK_INDEX=0     # worker-0 的索引
VC_TASK_INDEX=0     # 同上
```

### 域名解析规则

```
# svc plugin 创建的 Headless Service
{job-name}.{namespace}.svc.cluster.local

# 单个 Pod 域名
{task-name}-{index}.{job-name}.{namespace}.svc.cluster.local

# 例如
worker-0.pytorch-distributed-training.default.svc.cluster.local
worker-1.pytorch-distributed-training.default.svc.cluster.local
worker-2.pytorch-distributed-training.default.svc.cluster.local
worker-3.pytorch-distributed-training.default.svc.cluster.local
```

---

## 7. 对比分析

### 7.1 Volcano vs 原生 Kubeflow Training Operator

| 特性 | Volcano | Kubeflow Training Operator (PyTorchJob) |
|------|---------|----------------------------------------|
| **核心能力** | 通用调度器（资源调度为主） | 专有 Operator（训练编排为主） |
| **Gang Scheduling** | ✅ 原生支持 | ⚠️ 需结合 Volcano 使用 |
| **多框架** | TF/PyTorch/MPI/Spark/... | TF/PyTorch/MPI/XGBoost |
| **资源管理** | Queue + 权重 + DRF | 依赖外部调度器 |
| **弹性训练** | 需用户手动配置 | 支持 TorchElastic |
| **环境注入** | svc/env plugin | 自动注入框架所需变量 |
| **推荐用法** | **Volcano + torchrun** | PyTorchJob + Volcano（叠加） |

### 7.2 模式对比

| 模式 | 优点 | 缺点 | 推荐场景 |
|------|------|------|----------|
| **A: 分角色**（显式 node_rank） | 角色清晰，运维直觉 | nnodes 硬编码，扩展性差 | 固定节点集群 |
| **B: 统一角色 + c10d** | 配置简洁，扩展性好 | 需要 k8s 域名发现支持 | **生产推荐** |
| **C: 弹性训练** | 动态扩缩容，资源利用率高 | 需要 checkpoint 支持 | 共享集群 |

---

## 8. 最佳实践总结

1. **使用 `torchrun` 而非 `torch.distributed.launch`** — 新 API，自动注入环境变量，支持容错
2. **推荐统一角色模式（B）** — 所有节点运行相同 torchrun 命令，c10d 自动协调
3. **`minAvailable` 必须等于总 Pod 数** — 确保 Gang Scheduling 生效
4. **启用 `env` + `svc` plugin** — 自动注入环境变量和域名解析
5. **`$(VC_MASTER_HOST)` 作为 rendezvous 端点** — Volcano 自动提供第一个 Pod 域名
6. **大容量 `/dev/shm`** — NCCL 通信需要，内存至少 64Gi
7. **NCCL 网络配置** — 设置 `NCCL_SOCKET_IFNAME`、`NCCL_DEBUG=INFO` 便于排障
8. **定期 checkpoint 到 PVC** — 容错和弹性训练的基础
9. **使用 Queue 做资源隔离** — 多团队共享集群时必备
10. **部署顺序** — Volcano → Queue → PVC → Job

---

## 9. 参考文档

- [PyTorch Distributed Overview](https://pytorch.org/tutorials/beginner/dist_overview.html)
- [torchrun (Elastic Launch) 官方文档](https://pytorch.org/docs/stable/elastic/run.html)
- [Volcano 官方文档](https://volcano.sh/en/docs/)
- [Volcano Job CRD](https://volcano.sh/en/docs/vcjob/)
- [Volcano Env Plugin](https://volcano.sh/en/docs/user-guide/how_to_use_env_plugin/)
- [Kubernetes AI Training Pipeline: Volcano + Training Operator + Kueue](https://www.youngju.dev/blog/kubernetes/ai_training_pipeline_k8s.en)
- [Batch AI Workloads with Volcano on K8s](https://kubernetes.recipes/recipes/ai/ai-batch-processing-volcano/)

---

> **总结**：PyTorch 的 Master/Worker 模式本质上是一个去中心化的 Worker 集合，通过 `MASTER_ADDR:MASTER_PORT` 进行协调初始化。Volcano 通过 Gang Scheduling 确保所有 Worker Pod 同时被调度，通过 svc plugin 提供 DNS 发现，通过 env plugin 注入索引。最佳实践是使用 `torchrun` + c10d rendezvous + Volcano 统一角色模式，实现简洁、可扩展、容错的生产级分布式训练。
