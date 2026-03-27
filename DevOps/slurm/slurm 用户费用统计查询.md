首先是 slurm.conf 需要做如下配置:

```sh
# 资源计费相关
PriorityWeightTRES=CPU=1000,Mem=2000,GRES/gpu=3000
AccountingStorageTRES=gres/gpu:GTX1050Ti,gres/gpu:GTX1060,gres/gpu:GTX1660Super,gres/npu:Ascend910,gres/ascend:Ascend910
```

然后更新系统配置,使用以下指令:

```sh
scontrol reconfigure
```

使用一下指令来查看计费情况:

```sh
sacct -u 用户名 -j 作业名 -l
```

使用 format 有以下配置选择:

```sh
Account             AdminComment        AllocCPUS           AllocNodes
AllocTRES           AssocID             AveCPU              AveCPUFreq
AveDiskRead         AveDiskWrite        AvePages            AveRSS
AveVMSize           BlockID             Cluster             Comment
Constraints         ConsumedEnergy      ConsumedEnergyRaw   Container
CPUTime             CPUTimeRAW          DBIndex             DerivedExitCode
Elapsed             ElapsedRaw          Eligible            End
ExitCode            FailedNode          Flags               GID
Group               JobID               JobIDRaw            JobName
Layout              MaxDiskRead         MaxDiskReadNode     MaxDiskReadTask
MaxDiskWrite        MaxDiskWriteNode    MaxDiskWriteTask    MaxPages
MaxPagesNode        MaxPagesTask        MaxRSS              MaxRSSNode
MaxRSSTask          MaxVMSize           MaxVMSizeNode       MaxVMSizeTask
McsLabel            MinCPU              MinCPUNode          MinCPUTask
NCPUS               NNodes              NodeList            NTasks
Partition           Planned             PlannedCPU          PlannedCPURAW
Priority            QOS                 QOSRAW              Reason
ReqCPUFreq          ReqCPUFreqGov       ReqCPUFreqMax       ReqCPUFreqMin
ReqCPUS             ReqMem              ReqNodes            ReqTRES
Reservation         ReservationId       Start               State
Submit              SubmitLine          Suspended           SystemComment
SystemCPU           Timelimit           TimelimitRaw        TotalCPU
TRESUsageInAve      TRESUsageInMax      TRESUsageInMaxNode  TRESUsageInMaxTask
TRESUsageInMin      TRESUsageInMinNode  TRESUsageInMinTask  TRESUsageInTot
TRESUsageOutAve     TRESUsageOutMax     TRESUsageOutMaxNode TRESUsageOutMaxTask
TRESUsageOutMin     TRESUsageOutMinNode TRESUsageOutMinTask TRESUsageOutTot
UID                 User                UserCPU             WCKey
WCKeyID             WorkDir
```

# 字段解析

### 全部 ALL

```
  打印下面列出的所有字段。
```

### 帐户 Account

```
  作业运行的帐户。
```

### 管理员评论 AdminComment

```
  作业的注释字符串，必须由管理员、SlurmUser 或 root 设置。
```

### 分配CPU AllocCPUs

```
  分配的 CPU 计数。相当于NCPUS。
```

### 分配节点 AllocNodes

```
  分配给作业/步骤的节点数。如果作业待处理，则为 0。
```

### 分配可追溯资源 AllocTres

```
  可追踪的资源。这些是作业开始运行后分配给作业/步骤的资源。对于待处理的作业，该字段应为空。有关更多详细信息，请参阅 slurm.conf 中的 AccountingStorageTRES。
  
  注意：当使用 no_consume 标志配置通用资源时，分配将打印为零。
```

### 关联ID AssocID

```
  参考用户、账户和集群的关联。
```

### 平均CPU AveCPU

```
  作业中所有任务的平均（系统+用户）CPU 时间。
```

### 平均CPU频率 AveCPUFreq

```
  作业中所有任务的平均加权 CPU 频率，以 kHz 为单位。
```

### 平均磁盘读取 AveDiskRead

```
  作业中所有任务读取的平均字节数。
```

### 平均磁盘写入 AveDiskWrite

```
  作业中所有任务写入的平均字节数。
```

### 平均页 AvePages

```
  作业中所有任务的平均页面错误数。
```

### 平均RSS AveRSS

```
  Average resident set size of all tasks in job.
  作业中所有任务的平均驻留集大小。
```

### 平均虚拟内存大小 AveVMSize

```
  作业中所有任务的平均虚拟内存大小。
```

### 区块ID BlockID

```
  要使用的块的名称（与 Blue Gene 系统一起使用）。
```

### 集群 Cluster

```
  集群名称。
```

### 备注 Comment

```
当slurm.conf文件中的AccountingStoreFlags参数包含“job_comment”时，作业的注释字符串。可以通过调用sacctmgr modify job或专门的sjobexitmod命令来修改Comment字符串。
```

### 约束条件 Constraints

```
  将请求的作业作为约束。
```

### 消耗能量 ConsumedEnergy

```
  工作中所有任务消耗的总能量（以焦耳为单位）。值可以包括单位前缀（K、M、G、T、P）。注：仅在独占作业分配的情况下，该值才反映作业的实际能耗。
```

### 消耗能量的Raw ConsumedEnergyRaw

```
  工作中所有任务消耗的总能量（以焦耳为单位）。注：仅在独占作业分配的情况下，该值才反映作业的实际能耗。
```

### 容器 Container

```
  请求的 OCI 容器捆绑包的路径。
```

### 中央处理器时间 CPUTime

```
  作业或步骤使用的时间（已用时间 * CPU 计数），格式为 HH:MM:SS。
```

### CPU时间RAW CPUTimeRAW

```
  作业或步骤使用的时间（已用时间 * CPU 计数）（以 cpu 秒为单位）。
```

### 数据库索引 DBIndex

```
  作业表中条目的唯一数据库索引。
```

### 退出代码 DerivedExitCode

```
  作业的作业步骤（srun 调用）返回的最高退出代码。冒号后面是导致进程终止的信号（如果进程是由信号终止的）。可以通过调用sacctmgr 修改作业 或专门的sjobexitmod命令来修改 DerivedExitCode 。
```

### 占用时间 Elapsed

```
  作业已经过去了。
  该字段的输出格式如下：
    [DD-[HH:]]MM:SS
  定义如下：
    DD  天
    HH  小时
    MM  分钟
    SS  秒
```

### 占用时间原始数据 ElapsedRaw

```
  作业的运行时间以秒为单位。
```

### 备条件的运行时间 Eligible

```
  当该工作要求条件满足的时间。格式与End相同。
```

### 作业结束时间 End

```
  工作终止时间。输出格式为 YYYY-MM-DDTHH:MM:SS，除非通过 SLURM_TIME_FORMAT 环境变量进行更改。
```

### 退出代码 ExitCode

```
  作业脚本或 salloc 返回的退出代码，通常由 exit() 函数设置。冒号后面是导致进程终止的信号（如果进程是由信号终止的）。
```

### 额外的 Extra

```
  当slurm.conf文件中的AccountingStoreFlags参数包含“job_extra”时，作业的额外字符串。可以通过调用sacctmgr modify job命令来修改Extra字符串。。
```

### 故障节点 FailedNode

```
  其故障导致作业被终止的节点的名称。
```

### 标志 Flags

```
  作业标志。当前标志是 SchedSubmit、SchedMain、SchedBackfill。
```

### GID

```
  运行作业的用户的组标识符。
```

### 组 Group

```
  运行作业的用户的组名。
```

### 作业ID JobID

```
  作业或作业步骤的标识号。
  常规工作的形式为：
    作业 ID[.JobStep]
  数组作业的形式为：
    ArrayJobID_ArrayTaskID
  异构工作的形式如下：
    HetJobID+HetJobOffset
  打印作业阵列时，如果指定单个作业 ID，则对于具有大量作业的系统，可以显着提高命令的性能。默认情况下，该字段大小限制为 64 字节。使用环境变量 SLURM_BITSTR_LEN 指定更大的字段大小。
```

### 作业IDRaw JobIDRaw

```
  作业或作业步骤的标识号。以JobID[.JobStep]形式打印常规作业、异构作业和数组作业的 JobID 。
```

### 作业名称 JobName

```
  作业或作业步骤的名称。slurm_accounting.log文件是一个以空格分隔的文件。因此，如果作业名中使用了空格，则在将记录写入记帐文件之前，会用下划线替换该空格。因此，当sacct显示作业名称时，其中包含空格的作业名称现在将用下划线代替空格。
```

### 布局 Layout

```
  步骤运行时的布局是什么。这可以让您了解哪个节点在您的作业中运行哪个级别。
```

### 最大磁盘读取 MaxDiskRead

```
  作业中所有任务读取的最大字节数。
```

### 最大磁盘读取节点 MaxDiskReadNode

```
  发生 maxdiskread 的节点。
```

### 最大磁盘读取任务 MaxDiskReadTask

```
  发生 maxdiskread 的任务 ID。
```

### 最大磁盘写入 MaxDiskWrite

```
  作业中所有任务写入的最大字节数。
```

### 最大磁盘写入节点 MaxDiskWriteNode

```
  发生 maxdiskwrite 的节点。
```

### 最大磁盘写入任务 MaxDiskWriteTask

```
  发生 maxdiskwrite 的任务 ID。
```

### 最大页数 MaxPages

```
  作业中所有任务的最大页面错误数。
```

### 最大页数节点 MaxPagesNode

```
  发生 maxpages 的节点。
```

### 最大页数任务 MaxPagesTask

```
  发生 maxpages 的任务 ID。
```

### 最大驻留集 MaxRSS

```
  作业中所有任务的最大常驻集大小。
```

### 最大驻留集节点 MaxRSSNode

```
  发生 maxrss 的节点。
```

### 最大驻留集任务 MaxRSSTask

```
  发生 maxrss 的任务 ID。
```

### 最大虚拟内存大小 MaxVMSize

```
  作业中所有任务的最大虚拟内存大小。
```

### 最大虚拟内存节点 MaxVMSizeNode

```
  发生 maxvmsize 的节点。
```

### 最大虚拟内存任务 MaxVMSizeTask

```
  发生 maxvmsize 的任务 ID。
```

### MCS标签 MCSLabel

```
  与作业关联的多类别安全 (MCS) 标签。在 slurm.conf 中启用 MCSPlugin 时添加到作业。
```

### 最小CPU MinCPU

```
  作业中所有任务的最小（系统+用户）CPU 时间。
```

### 最小CPU节点 MinCPUNode

```
  mincpu 发生的节点。
```

### 最小CPU任务 MinCPUTask

```
  mincpu 发生的任务 ID。
```

### NCPUS

```
  分配给作业的 CPU 总数。相当于AllocCPUS。
```

### NNodes

```
  作业或步骤中的节点数。如果作业正在运行或已运行，则此计数将是分配的数量，否则该数量将是请求的数量。
```

### 节点列表 NodeList

```
  作业/步骤中的节点列表。
```

### NTasks

```
  作业或步骤中的任务总数。
```

### 分区 Partition

```
  标识运行作业的分区。
```

### 计划 Planned

```
  这项工作的计划时间使用了多少挂钟时间。这是根据作业从合格时间到开始或取消的等待时间得出的。格式与Elapsed相同。
```

### 计划CPU PlannedCPU

```
  该作业按照计划时间使用了多少 CPU 秒。格式与Elapsed相同。
```

### 计划CPURAW PlannedCPURAW

```
  该作业按照计划时间使用了多少 CPU 秒。格式以处理器秒为单位。
```

### 优先权 Priority

```
  slurm 中的优先权。
```

### 服务质量 QOS

```
  服务质量名称。
```

### 服务质量RAW QOSRAW

```
服务质量的数字id。
```

### 原因 Reason

```
  作业因优先级或资源以外的原因而被阻止运行的最后一个原因。即使作业运行完成，这也会保存在数据库中。
```

### 请求CPU频率 ReqCPUFreq

```
  该步骤请求的 CPU 频率，以 kHz 为单位。注意：该值仅适用于作业步骤。没有报告该作业的价值。
```

### 请求CPU频率Gov ReqCPUFreqGov

```
  该步请求的 CPU 频率调节器，以 kHz 为单位。注意：该值仅适用于作业步骤。没有报告该作业的价值。
```

### 请求CPU频率最大 ReqCPUFreqMax

```
  该步骤请求的最大 CPU 频率，以 kHz 为单位。注意：该值仅适用于作业步骤。没有报告该作业的价值。
```

### 请求CPU频率最小值  ReqCPUFreqMin

```
  该步骤请求的最低 CPU 频率，以 kHz 为单位。注意：该值仅适用于作业步骤。没有报告该作业的价值。
```

### 请求CPUS ReqCPUS

```
  请求的 CPU 数量。
```

### 请求内存 ReqMem

```
  作业所需的最小内存。它可能会附加一个字母来指示单位（M 表示兆字节，G 表示千兆字节，等等）。注意：该值仅来自作业分配，而不是步骤。
```

### 请求节点 ReqNodes

```
  请求的作业/步骤的最小节点数。
```

### 请求可追溯资源 ReqTres

```
  可追踪的资源。这些是作业/步骤在提交时请求的最小资源计数。有关更多详细信息，请参阅 slurm.conf 中的 AccountingStorageTRES。
```

### 预订 Reservation

```
  预订名称。
```

### 预订ID ReservationId

```
  预订编号
```

### 作业开始时间 Start

```
  作业的启动时间。格式与End相同。
```

### 作业状态 State

```
  显示作业状态或状态。有关可能状态的列表， 请参阅下面的作业状态代码部分。
  如果有关作业状态的可用信息多于当前字段宽度所容纳的信息（例如，取消作业的 UID），则状态后将跟有“+”。您可以使用前面描述的“%NUMBER”格式修饰符来增加显示状态的大小。
  注意：RUNNING 状态也将返回挂起的作业。为了打印挂起的作业，您必须在与 RUNNING 不同的调用中请求 SUSPENDED。
  注意：RUNNING 状态将返回在请求的时间段内完成的任何作业（取消或以其他方式），因为该作业在此期间也在 RUNNING 中。如果您仅查找已完成的作业，请选择适当的状态，而不选择“正在运行”状态。
```

### 作业提交时间 Submit

```
  提交作业的时间。格式与End相同。
  注意：如果作业重新排队，提交时间将重置。要获取原始提交时间，需要使用 -D 或 --duplicate 选项来显示作业的所有重复条目。
```

### 提交作业指令 SubmitLine

```
  发出提交作业的完整命令。
```

### 作业暂停时长 Suspended

```
  作业或作业步骤暂停的时间量。格式与Elapsed相同。
```

### 系统备注 SystemComment

```
  作业的注释字符串通常由插件设置。只能由 Slurm 管理员修改。
```

### 系统CPU SystemCPU

```
  作业或作业步骤使用的系统 CPU 时间量。格式与Elapsed相同。
  注意：有关如何处理取消的作业的信息，请参阅 TotalCPU 的注释。
```

### 时限 Timelimit

```
  该工作的时限是多少。格式与Elapsed相同。
```

### 时间限制原始 TimelimitRaw

```
该工作的时限是多少。格式为分钟数。
```

### 总CPU数 TotalCPU

```
  作业或作业步骤使用的 SystemCPU 和 UserCPU 时间的总和。对于包含多个作业步骤的作业，作业的总 CPU 时间可能会超过作业的运行时间。格式与Elapsed相同。
  注意：对于被信号中断的步骤（例如 scancel、作业超时），TotalCPU 提供了任务父进程的度量，可能不包括子进程的 CPU 时间。这是wait3资源使用 ( getrusage ) 内部结构的结果。对于以常规方式完成的进程，所有后代进程（fork 和 exec）资源都包括在内。但是，如果进程被终止，proctrack 插件和最终用户应用程序之间的结果可能会有所不同。
```

### Tres平均使用量 TresUsageInAve

```
  Tres 工作中所有任务的平均使用率。 注意：如果相应的 TresUsageInMaxTask 为 -1，则该指标以节点为中心而不是以任务为中心。
```

### Tres最大使用量  TresUsageInMax

```
  工作中所有任务的最大使用量。 注意：如果相应的 TresUsageInMaxTask 为 -1，则该指标以节点为中心而不是以任务为中心。
```

### Tres最大使用量节点 TresUsageInMaxNode

```
  发生每个最大 TRES 使用情况的节点。
```

### Tres 最大使用量节点 TresUsageInMaxTask

```
  发生每个最大 TRES 使用量的任务。
```

### Tres最小使用量 TresUsageInMin

```
  工作中所有任务的最小使用量。 注意：如果相应的 TresUsageInMinTask 为 -1，则该指标以节点为中心而不是以任务为中心。
```

### Tres最小使用量节点 TresUsageInMinNode

```
  发生每个最小 TRES 使用情况的节点。
```

### Tres最小使用量任务 TresUsageInMinTask

```
  发生每个最小 TRES 使用量的任务。
```

### Tres总使用量 TresUsageInTot

```
  Tres 工作中所有任务的总使用量。
```

### Tres的平均使用率 TresUsageOutAve

```
  Tres 工作中所有任务的平均使用率。 注意：如果相应的 TresUsageOutMaxTask 为 -1，则该指标以节点为中心而不是以任务为中心。
```

### Tres 最大使用量 TresUsageOutMax

```
  Tres 工作中所有任务的最大使用量。 注意：如果相应的 TresUsageOutMaxTask 为 -1，则该指标以节点为中心而不是以任务为中心。
```

### Tres使用最大次数节点 TresUsageOutMaxNode

```
  发生每个最大 TRES 使用情况的节点。
```

### Tres使用最大使用量 TresUsageOutMaxTask

```
  发生每个最大 TRES 使用量的任务。
```

### Tres使用最小使用量 TresUsageOutMin

```
  Tres 工作中所有任务的最低使用率。
```

### Tres使用最小次数节点 TresUsageOutMinNode

```
  发生每个最小 TRES 使用情况的节点。
```

### Tres使用最小次数任务 TresUsageOutMinTask

```
  发生每个最小 TRES 使用量的任务。
```

### Tres使用总次数 TresUsageOutTot

```
  Tres 工作中所有任务的总使用量。
```

### UID

```
  运行作业的用户的用户标识符。
```

### 用户 User

```
  运行作业的用户的用户名。
```

### 用户CPU UserCPU

```
  作业或作业步骤使用的用户 CPU 时间量。格式与Elapsed相同 。
  注意：有关如何处理取消的作业的信息，请参阅 TotalCPU 的注释。
```

### wckey密钥 WCKey

```
  工作负载特征键。用于将正交帐户分组在一起的任意字符串。
```

### WC密钥ID WCKeyID

```
  参考wckey。
```

### 工作目录  WorkDir

```
  作业执行命令所使用的目录。
```
