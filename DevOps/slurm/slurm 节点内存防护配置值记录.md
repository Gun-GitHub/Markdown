# slurm 节点内存防护配置值记录

> 场景：节点约 100G 内存，两个 50G 作业把物理内存打满 → 整机 OOM → 内核 OOM killer 随机杀 sshd/slurmd → 节点假死、Slurm 无法杀作业。
> 思路：让 Slurm 从"软记账"变为"硬隔离"——调度器预留系统内存 + cgroup 给作业套内核级内存硬限制，超限由内核毫秒级 SIGKILL 该作业自己的 cgroup，整机永不 OOM。

## 核心配置值速查

| 参数 | 配置值 | 所在文件 | 作用层面 |
|---|---|---|---|
| `MemSpecLimit` | `8192`（= 8G，**纯 MiB 数字**） | slurm.conf（NodeName 行） | 调度器：节点上保留 8G 给系统，不分配给作业 |
| `ConstrainRAMSpace` | `yes` | cgroup.conf | 执行层：开关——给作业 cgroup 设内存硬限制 |
| `AllowedRAMSpace` | `100` | cgroup.conf | 执行层：硬限制 = 申请量 × 100% |
| `SelectType` | `select/cons_tres` | slurm.conf | 启用 TRES 多维资源分配（Memory 参与调度） |
| `SelectTypeParameters` | `CR_Core_Memory` | slurm.conf | 内存 TRES 参与调度（MemSpecLimit 生效前提） |
| `TaskPlugin` | `task/cgroup,task/affinity` | slurm.conf | 作业进程放入 cgroup |
| `JobAcctGatherType` | `jobacct_gather/cgroup` | slurm.conf | 按 cgroup 统计，数据更准 |
| `CgroupPlugin` | `cgroup/v2`（按系统实际版本） | cgroup.conf | cgroup 版本 |
| `OOMKillStep` | `yes` | slurm.conf | cgroup OOM 杀 step 内任一任务 → 终止整个 step |
| `KillWait` | `1` | slurm.conf | 时限/取消后 SIGTERM → SIGKILL 等待秒数（默认 30） |
| `SlurmdTimeout` | `60` | slurm.conf | 节点失联更快标记 DOWN（默认 300s） |
| `PrologFlags` | `Contain` | slurm.conf | 节点作业清空后回收 cgroup 资源 |
| `ConstrainSwapSpace` | `yes`（建议） | cgroup.conf | 限制 swap，防作业逃进 swap 拖慢系统 |

## 三个关键值详解

### 1. MemSpecLimit=8192 —— 调度器视角的"预留"

- 官方定义：Amount of RealMemory, **in mebibytes**, reserved for system use and not available for user allocations. Must be less than RealMemory.
- **单位坑：只支持纯数字（MiB）和 K 后缀，`M`/`G`/`T` 都不认。**
  - 源码依据：`src/common/parse_value.c` 的 `s_p_handle_uint64()` 中 `strtoull` 之后仅处理 `k`/`K` 后缀，其余字符（含 `G`）直接报 `value (8G) is not a valid number`，该键被**静默丢弃**。
  - ❌ `MemSpecLimit=8G` → 配置不生效，slurmctld 日志有报错
  - ✅ `MemSpecLimit=8192`
- 生效两件事：
  1. 调度视角：节点可分配 = RealMemory − MemSpecLimit。100G 节点（102400 MiB）留 8G 后，作业最多拿到 94208 MiB ≈ 92G；
  2. cgroup 视角（配 task/cgroup 时）：slurmd 守护进程被放进限额 8G 的 cgroup，且**该 cgroup 禁用 OOM killer**——超限只会自己卡住，绝不被内核杀，保证 Slurm 存活。
- 生效前提：
  - `SelectTypeParameters` 含 Memory TRES（`CR_Core_Memory`），否则不工作；
  - 值必须**严格小于** RealMemory（源码有检查，超了重置为 0 并报错）。
- 官方提醒：NodeName 里的 RealMemory 要和 `slurmd -C` 报告的实际内存一致。

### 2. ConstrainRAMSpace=yes —— 从"记账"变"执法"

- 官方定义：If configured to "yes" then constrain the job's RAM usage by setting the memory **soft limit to the allocated memory** and the **hard limit to the allocated memory × AllowedRAMSpace**.
- 默认 `no` —— **no 就是节点卡死的根源**：Slurm 只在账本上记申请量，作业实际多吃没人管，整机 OOM，内核 OOM killer 全系统乱杀。
- 设为 `yes` 后：每个作业独立 memory cgroup，内核强制执行。
  - 作业 A 超限 → **内核触限瞬间对作业 A cgroup 内进程 SIGKILL**，毫秒级，不走轮询；
  - 作业 B 毫发无伤（独立 cgroup）；
  - 整机内存被 cgroup 兜底，**到不了整机 OOM**，节点不假死、可登录。
- 关键 NOTE：触发 OOM 时内核只杀该 step 的**一个或多个**进程，step 本身还活着、其余进程继续跑 → 多进程作业必须配 `OOMKillStep`，一有任务被 OOM 就终止整个 step 的所有节点，防孤儿进程。
  - 源码链路：`task_cgroup_memory.c` 中 `oom_kill_cnt > 0 && oom_kill_step` → `slurm_terminate_job_step()`。
- 代价：官方提醒启用后**单节点作业吞吐会有可感知下降**（每次内存分配都走 cgroup 记账），拿吞吐换安全，值得。

### 3. AllowedRAMSpace=100 —— 硬限制比例系数

- 官方定义：Constrain the job/step cgroup RAM to this percentage of the allocated memory. soft limit = 申请量，hard limit = 申请量 × AllowedRAMSpace/100.
- 申请 50G 的作业实际得到（=100 时）：

| 限制 | 值 | 含义 |
|---|---|---|
| soft limit | 50G | 内存紧张时内核优先从该 cgroup 回收 |
| hard limit | 50G | **绝对上限，超过即 cgroup OOM → SIGKILL** |

- 取值区间行为（文档原文归纳）：
  - **= 100**：作业恰好拿到申请量，语义最干净，**推荐**；
  - **> 100**：允许超申请量使用，文档警告 *may cause system Out Of Memory events* —— 等于把问题请回来，别用；
  - **< 100**（如 90）：实际拿到的比申请少，留安全边际但用户"申请 50G 却跑不过 45G"，体验差，一般不这么干。
- 是 cgroup.conf 的**全局默认值**，作业级可用 `--mem` 覆盖。

## 场景账本：100G 节点 + 两个 50G 作业

```
RealMemory  = 102400 MiB
MemSpecLimit= 8192 MiB
可分配      = 94208 MiB ≈ 92G
50G + 50G   = 102400 MiB > 94208
→ 第二个 50G 作业直接 pending（调度器拒绝），挤不上去
```

- 以前：两个 50G 挤上去 → 互相拖死。现在：账面上就放不下。
- 若确需两个作业共节点：调小预留，或让作业诚实申请（如各 45G），总得为 OS 让路。

两个 46G 作业（共 92G 打满可分配）时的防护链：

1. 作业 A 某进程偷偷申请到 52G → 内核当场 SIGKILL 该进程（`dmesg` 有 cgroup OOM 记录，slurmstepd 日志记 OOM 事件）；
2. `OOMKillStep=yes` → 作业 A 整体终止，作业 B 继续跑；
3. OS 手里 8G 预留 → sshd 正常、可登录、slurmctld 正常记账；
4. 节点**全程不假死**。

## 推荐落地配置

```ini
# slurm.conf（节点级）
SelectType=select/cons_tres
SelectTypeParameters=CR_Core_Memory
TaskPlugin=task/cgroup,task/affinity
JobAcctGatherType=jobacct_gather/cgroup
PrologFlags=Contain
OOMKillStep=yes                # 需 cgroup v2 + 内核 >4.19
KillWait=1
SlurmdTimeout=60
NodeName=... RealMemory=102400 MemSpecLimit=8192
```

```ini
# cgroup.conf
CgroupPlugin=cgroup/v2         # 按系统实际版本，v1 亦可
ConstrainRAMSpace=yes
AllowedRAMSpace=100
ConstrainSwapSpace=yes
```

配套动作（文档点名，别漏）：

- 节点定义与实际内存一致：以 `slurmd -C` 为准；
- 建议启用 `pam_slurm_adopt`，否则用户 ssh 登录后开的进程在 cgroup 外，能绕开限制。

## 验证方法

改完 `scontrol reconfigure` 后：

1. 跑测试作业，查限额是否落下：
   - cgroup v2：`cat /sys/fs/cgroup/.../slurm/step_*/memory.max`（应 = 申请量字节数）
   - cgroup v1：`cat .../memory.limit_in_bytes`
2. 故意超限：申请 1G 跑 `stress --vm 1 --vm-bytes 3G`
   - 预期：几秒内进程被 kill；
   - 预期：节点 `free -g` 里 OS 预留纹丝不动。

## 相关参数备注

- `KillWait`：作业到时限/取消时，Slurm 先发 SIGTERM，等 `KillWait` 秒未退出再发 SIGKILL。默认 30，最大 65533。只处理时限/取消路径，**不处理内存 OOM**。
- `SlurmdTimeout=60`：节点失联时更快被 slurmctld 标记 DOWN（默认 300s），缩短假死后摘节点的时间。
- `JobAcctGatherParameters=OverMemoryKill`：无 cgroup 硬限制时的降级方案，轮询检测超量杀 step，有延迟，不如 cgroup 即时。
