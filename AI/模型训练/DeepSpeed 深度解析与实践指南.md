[[模型训练]]
> **一句话定义**：DeepSpeed 是微软开源的一套深度学习优化库，旨在让大模型训练变得"更快、更大、更省"，是当前大模型（LLM）训练领域事实上的标准工具。

---

## 1. 这是什么？

DeepSpeed 是由 **微软** 开发的、运行在 PyTorch 之上的分布式训练优化框架。

它不是一个独立的深度学习框架（比如你不能只靠 DeepSpeed 搭建模型），而是**PyTorch 的增强插件**——你仍然用 PyTorch 定义模型和数据集，但将模型引擎交给 DeepSpeed 来驱动，由它来管理分布式通信、显存优化和训练加速。

你可以把它想象成一辆汽车的**涡轮增压系统**：发动机本身还是那台发动机（PyTorch），但加上涡轮增压（DeepSpeed）后，同样的排量能爆发出远超原厂的动力。

---

## 2. 用来干什么？

DeepSpeed 解决的核心问题非常明确：

### 2.1 模型太大了，单张 GPU 放不下

以 GPT-3 175B 为例，仅模型参数就占用约 **350GB** 显存（fp16）。最顶级的 A100 80GB 显存也塞不下。普通方法你连模型都加载不了。

DeepSpeed 通过 **ZeRO（零冗余优化器）** 将参数、梯度、优化器状态分散存储到所有参与训练的 GPU 上，让不可能变成可能。

### 2.2 训练太慢了，需要加速

通过通信优化（如梯度压缩、通信与计算重叠），DeepSpeed 能在相同的硬件条件下，让训练吞吐量提升数倍。

### 2.3 硬件不够用，想省钱

DeepSpeed 允许使用更多数量但性能较低的 GPU 来训练大模型，而不是非要 8 张 A100 不可。它可以在 4 张 3090（24GB）上微调原本需要 80GB 显存的模型。

---

## 3. 底层逻辑是什么？

DeepSpeed 的底层逻辑是三个方面：

### 3.1 ZeRO（Zero Redundancy Optimizer）——零冗余优化器

这是 DeepSpeed 的核心命脉。在传统 DDP（数据并行）中，每一张 GPU 都保存着完整的模型参数、梯度、优化器状态，存在大量冗余。

**ZeRO 的核心思想**：既然数据可以切片（Data Parallelism），那参数为什么不能切片？

| ZeRO 阶段 | 做了什么 | 显存节省量 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **ZeRO-1** | 优化器状态分片（Adam 动量等） | 约 4x | 入门级，小集群 |
| **ZeRO-2** | 优化器状态 + 梯度分片 | 约 8x | 中等规模 |
| **ZeRO-3** | 优化器状态 + 梯度 + **模型参数**都分片 | 约 N 倍（N=GPU 数量） | 超大模型跨多节点 |

**为什么叫"零冗余"**？因为在理想情况下，每张 GPU 只存储自己应得的那 1/N 份，冗余几乎为零。

但 ZeRO-3 的代价是通信开销——每次计算前需要从各个节点"借"回当前层需要的参数。这就是为什么 ZeRO 需要极其高效的通信引擎。

### 3.2 通信引擎 —— 同一套代码适配不同硬件

DeepSpeed 内置了一个经过高度优化的通信引擎，它能够：

- **自动选择通信后端**（NCCL 用于 NVIDIA GPU、GLOO 用于 CPU 场景）
- **梯度压缩**：通过 1-bit Adam、1-bit LAMB 等算法，将通信量压缩数倍
- **通信与计算重叠**：在反向传播计算当前层的梯度时，同时传输之前的梯度，隐藏通信延迟

### 3.3 显存优化技术

除了 ZeRO，DeepSpeed 还提供：

- **Activation Checkpointing（激活检查点）**：不保留中间激活值，反向传播时重新计算，以计算换显存
- **Offload（卸载）**：将优化器状态或参数卸载到 CPU 内存，进一步释放 GPU 显存（ZeRO-Offload）

---

## 4. 支持哪些并行策略？

DeepSpeed 几乎覆盖了所有主流的并行训练模式：

| 策略 | DeepSpeed 是否支持 | 说明 |
| :--- | :--- | :--- |
| **DDP（数据并行）** | ✅ | 基础能力，内置支持 |
| **ZeRO-1 / ZeRO-2 / ZeRO-3** | ✅ 核心特色 | 渐进式分片，灵活配置 |
| **TP（张量并行）** | ✅ （通过 Megatron-LM 融合） | 与 NVIDIA Megatron 深度融合 |
| **PP（流水线并行）** | ✅ | 支持手动和自动流水线切割 |
| **SP（序列并行）** | ✅ | 超长序列场景下的优化 |
| **MoE（混合专家模型）** | ✅ | 支持稀疏 MoE 训练 |

其中，ZeRO + TP + PP 的组合被称为 **3D Parallelism（三维并行）**，这是当前训练千亿级模型的标配方案。

---

## 5. 怎么安装？

### 5.1 最简单的安装方式（推荐）

```bash
# 通过 pip 安装
pip install deepspeed

# 如果要用到 ZeRO-Offload 等功能，需要安装完整版
pip install deepspeed[all]
```

### 5.2 验证安装

```bash
ds_report
```

这个命令会输出 DeepSpeed 的安装状态，包括 NCCL 版本、CUDA 版本、可用 GPU 等信息。

### 5.3 前提条件

- Python >= 3.8
- PyTorch >= 1.8（推荐 2.0+）
- CUDA >= 11.0
- 有效的 NVIDIA GPU 驱动
- 多机场景需要：**各节点之间能够通过 SSH 免密访问**（用于进程分发）

---

## 6. 怎么使用？

### 6.1 配置 DeepSpeed

DeepSpeed 的核心配置通过一个 JSON 文件提供（也可以直接通过 Python 字典传入）。

**最小 `ds_config.json` 示例：**

```json
{
  "train_batch_size": 32,
  "gradient_accumulation_steps": 2,
  "fp16": {
    "enabled": true
  },
  "zero_optimization": {
    "stage": 2
  },
  "optimizer": {
    "type": "AdamW",
    "params": {
      "lr": 3e-5,
      "weight_decay": 0.01
    }
  },
  "scheduler": {
    "type": "WarmupLR",
    "params": {
      "warmup_min_lr": 0,
      "warmup_max_lr": 3e-5,
      "warmup_num_steps": 500
    }
  }
}
```

### 6.2 修改训练代码

使用 DeepSpeed 改造现有 PyTorch 训练代码，核心改动只有一处：**将 `model` 交给 `deepspeed.initialize` 管理**。

**改造前（普通 PyTorch）：**

```python
model = MyModel()
optimizer = torch.optim.AdamW(model.parameters(), lr=3e-5)

for batch in dataloader:
    loss = model(batch)
    loss.backward()
    optimizer.step()
```

**改造后（DeepSpeed）：**

```python
import deepspeed

model = MyModel()

# 关键一行：deepspeed.initialize 接管模型和优化器
model_engine, optimizer, _, _ = deepspeed.initialize(
    args=None,
    model=model,
    model_parameters=model.parameters(),
    config="ds_config.json"  # 上面我们写的那个 JSON 文件
)

for batch in dataloader:
    loss = model_engine(batch)
    model_engine.backward(loss)
    model_engine.step()
```

### 6.3 单机启动训练

```bash
deepspeed train.py --deepspeed_config ds_config.json
```

DeepSpeed 会自动检测本地 GPU 数量并启动对应数量的进程。

### 6.4 多机多卡启动训练

需要准备 **Hostfile**（主机列表），告诉 DeepSpeed 有多少台机器：

```text
# hostfile 内容
192.168.1.10 slots=8   # 第一台机器，8 张 GPU
192.168.1.11 slots=8   # 第二台机器，8 张 GPU
192.168.1.12 slots=4   # 第三台机器，4 张 GPU（异构也是可以的）
```

然后启动：

```bash
deepspeed --hostfile hostfile train.py --deepspeed_config ds_config.json
```

DeepSpeed 会 SSH 登录到 hostfile 中的每一台机器，在各个机器上启动对应的进程。

### 6.5 完整的训练代码示例

下面是一个完整的、可直接运行的示例，使用 DeepSpeed 训练一个简单的 MLP 模型：

```python
import torch
import deepspeed
from torch.utils.data import DataLoader, Dataset

# ============ 1. 定义数据集 ============
class RandomDataset(Dataset):
    def __init__(self, size=1000, input_dim=1024):
        self.data = torch.randn(size, input_dim)
        self.labels = torch.randn(size, 1)
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

# ============ 2. 定义模型 ============
class SimpleMLP(torch.nn.Module):
    def __init__(self, input_dim=1024, hidden_dim=512):
        super().__init__()
        self.net = torch.nn.Sequential(
            torch.nn.Linear(input_dim, hidden_dim),
            torch.nn.ReLU(),
            torch.nn.Linear(hidden_dim, hidden_dim),
            torch.nn.ReLU(),
            torch.nn.Linear(hidden_dim, 1)
        )
    
    def forward(self, x):
        return self.net(x)

# ============ 3. 训练函数 ============
def train():
    # 创建模型
    model = SimpleMLP()
    
    # 初始化 DeepSpeed 引擎
    model_engine, optimizer, _, _ = deepspeed.initialize(
        args=None,
        model=model,
        model_parameters=model.parameters(),
        config="ds_config.json"
    )
    
    # 数据集
    dataset = RandomDataset()
    dataloader = DataLoader(dataset, batch_size=64, shuffle=True)
    
    print(f"DeepSpeed ZeRO stage: {model_engine.zero_optimization_stage()}")
    print(f"Total training steps: {len(dataloader)}")
    
    # 训练循环
    for epoch in range(5):
        total_loss = 0.0
        for data, labels in dataloader:
            data = data.to(model_engine.device)
            labels = labels.to(model_engine.device)
            
            outputs = model_engine(data)
            loss = torch.nn.functional.mse_loss(outputs, labels)
            
            model_engine.backward(loss)
            model_engine.step()
            
            total_loss += loss.item()
        
        avg_loss = total_loss / len(dataloader)
        print(f"Epoch {epoch+1}/5, Average Loss: {avg_loss:.6f}")

if __name__ == "__main__":
    train()
```

配套的 `ds_config.json`：

```json
{
  "train_batch_size": 64,
  "gradient_accumulation_steps": 1,
  "fp16": {
    "enabled": true
  },
  "zero_optimization": {
    "stage": 2
  },
  "optimizer": {
    "type": "AdamW",
    "params": {
      "lr": 1e-3,
      "weight_decay": 0.01
    }
  }
}
```

---

## 7. 能和 Volcano 结合使用吗？

**可以，而且是生产环境下的推荐方案。**

Volcano 解决了 DeepSpeed 在 Kubernetes 上运行的"最后一公里"问题。

### 7.1 核心问题

在 K8s 上运行 DeepSpeed 多机训练时，遇到的核心问题是：

- DeepSpeed 要求**所有节点同时就绪**才能开始训练（因为它需要所有进程同时加入 Rendezvous）
- 但 Kubernetes 原生调度器是逐个调度 Pod 的，后启动的 Pod 可能因为资源不足而一直处于 Pending 状态
- 结果：先启动的 Pod 占了 GPU 但无法训练，后启动的 Pod 等不到资源——**死锁**

这正是 Volcano 的 **Gang Scheduling（成组调度）** 要解决的。

### 7.2 Volcano 解决的核心问题

| 问题 | 没有 Volcano | 有 Volcano |
| :--- | :--- | :--- |
| 资源调度 | 逐个调度，部分 Pod 等待 | 组调度，要么全部启动，要么一个都不启 |
| 任务管理 | 需要手动编排多个 Pod | 通过 `Job` 资源统一管理 |
| 故障恢复 | Pod 失败无法自动重启 | 支持 Pod 级别的重启策略 |
| 资源依赖 | Pod 间无感知 | Gang 机制确保相互感知 |

### 7.3 结合使用的方法

DeepSpeed 和 Volcano 的结合架构如下：

```
┌───────────────────────────────────────────────────┐
│                    Volcano Job                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ Master   │  │ Worker-1 │  │ Worker-2 │         │
│  │ Task     │  │ Task     │  │ Task     │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│       │              │              │             │
│       └───────┬──────┴───────┬──────┘             │
│               │              │                    │
│          Gang Scheduling   minAvailable=3         │
└───────────────────────────────────────────────────┘
```

火山 Job 同时启动 Master Task（1 个副本）和 Worker Task（2 个副本）。只有三者在资源和时间上都准备好时，Volcano 才会一次性调度。

### 7.4 Volcano Job YAML 完整示例

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: deepspeed-training
spec:
  # 使用 Volcano 调度器
  schedulerName: volcano
  
  # Gang Scheduling 配置
  minAvailable: 3
  
  tasks:
    # ---------- Master 节点 ----------
    - replicas: 1
      name: master
      template:
        spec:
          containers:
            - name: deepspeed-worker
              image: my-registry/training-image:latest
              command:
                - deepspeed
                - --num_nodes=3
                - --master_addr=$(POD_IP)
                - train.py
                - --deepspeed_config=ds_config.json
              env:
                - name: POD_IP
                  valueFrom:
                    fieldRef:
                      fieldPath: status.podIP
                - name: NCCL_SOCKET_IFNAME
                  value: eth0
              resources:
                limits:
                  nvidia.com/gpu: 8
          restartPolicy: OnFailure
    
    # ---------- Worker 节点 ----------
    - replicas: 2
      name: worker
      template:
        spec:
          containers:
            - name: deepspeed-worker
              image: my-registry/training-image:latest
              command:
                - deepspeed
                - --num_nodes=3
                - train.py
                - --deepspeed_config=ds_config.json
              env:
                - name: MASTER_ADDR
                  value: master.deepspeed-training.svc.cluster.local
                - name: NCCL_SOCKET_IFNAME
                  value: eth0
              resources:
                limits:
                  nvidia.com/gpu: 8
          restartPolicy: OnFailure
```

**关键点解释**：

1. **`minAvailable: 3`**：要求所有 1 个 Master + 2 个 Worker 同时就绪才能调度
2. **Master 通过 `status.podIP`** 获取自己的 IP，Worker 通过固定 Service 名称访问 Master
3. **`restartPolicy: OnFailure`**：进程异常退出时自动重启
4. **NCCL 网络配置**：通过 `NCCL_SOCKET_IFNAME` 指定通信网卡，在多网卡环境中至关重要

---

## 8. 简单用例：在 Volcano 上跑一个完整的 DeepSpeed 训练

以下是一个端到端的简单用例，执行一个完整的 DeepSpeed ZeRO-2 训练任务。

### 8.1 前提条件

- 一个运行中的 Kubernetes 集群，安装了 Volcano
- 集群中的节点拥有 NVIDIA GPU
- 训练镜像已经构建好（可以参考下面的 Dockerfile）

### 8.2 构建训练镜像

```dockerfile
# Dockerfile
FROM pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime

RUN pip install deepspeed

COPY train.py /workspace/train.py
COPY ds_config.json /workspace/ds_config.json

WORKDIR /workspace
CMD ["deepspeed", "--num_nodes=2", "train.py", "--deepspeed_config=ds_config.json"]
```

构建并推送到镜像仓库：

```bash
docker build -t my-registry/deepspeed-demo:latest .
docker push my-registry/deepspeed-demo:latest
```

### 8.3 提交 Volcano Job

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: deepspeed-demo
spec:
  schedulerName: volcano
  minAvailable: 2
  
  tasks:
    - replicas: 1
      name: master
      template:
        spec:
          containers:
            - name: train
              image: my-registry/deepspeed-demo:latest
              command:
                - deepspeed
                - --num_nodes=2
                - --master_addr=$(POD_IP)
                - train.py
                - --deepspeed_config=ds_config.json
              env:
                - name: POD_IP
                  valueFrom:
                    fieldRef:
                      fieldPath: status.podIP
              resources:
                limits:
                  nvidia.com/gpu: 4
          restartPolicy: OnFailure
    
    - replicas: 1
      name: worker
      template:
        spec:
          containers:
            - name: train
              image: my-registry/deepspeed-demo:latest
              command:
                - deepspeed
                - --num_nodes=2
                - train.py
                - --deepspeed_config=ds_config.json
              env:
                - name: MASTER_ADDR
                  value: master.deepspeed-demo.svc.cluster.local
              resources:
                limits:
                  nvidia.com/gpu: 4
          restartPolicy: OnFailure
```

### 8.4 提交并观察

```bash
# 提交任务
kubectl apply -f deepspeed-volcano-job.yaml

# 查看 Job 状态
kubectl get job -n volcano-system

# 查看 Pod 状态（当所有 Pod 同时变成 Running 时，说明 Gang Scheduling 生效了）
kubectl get pods -l job-name=deepspeed-demo -w

# 查看训练日志
kubectl logs -l job-name=deepspeed-demo -f
```
