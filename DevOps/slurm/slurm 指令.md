[[slurm]]
# [**主要参考**](https://docs.slurm.cn/master/man-pages-shou-ce)

# slurm 用户命令包括:

|序号  |指令             |解释|
|:--:|:--:|:--|
|1|sacct|用于报告关于活动或已完成的工作或工作步骤的计费信息。|
|2|sacctmgr|用于查看和修改 slurm 帐户信息。|
|3|salloc|用于实时为作业分配资源。这通常用于分配资源和生成 shell。然后使用 shell 执行 srun 命令来启动并行任务。|
|4|sattach|用于将标准输入、输出和错误加信号功能附加到当前正在运行的作业或作业步骤。可以多次附加到作业并从作业分离。|
|5|sbatch|用于提交作业脚本，以便以后执行。该脚本通常包含一个或多个用于启动并行任务的 srun 命令。|
|6|sbcast|用于将文件从本地磁盘传输到作业运行节点的本地磁盘。这可以用来有效地使用无磁盘计算节点，或者相对于共享文件系统提供更好的性能。|
|7|scancel|用于取消挂起的、正在运行的作业或作业步骤。它还可以用来向与正在运行的作业或作业步骤相关联的所有进程发送任意信号。|
|8|scontrol|是用于查看和/或修改 slurm 状态的管理工具。注意，许多 scontrol 命令只能以用户 root 身份执行。|
|9|scrontab|管理 slurm crontab 文件。|
|10|sdiag|调度诊断工具。|
|11|sh5util|用于 acct_together_profile 插件的合并实用程序。|
|12|sinfo|报告由 slurm 管理的分区和节点的状态。它有各种各样的筛选、排序和格式化选项。|
|13|sprio|用于显示影响作业优先级的组件的详细视图。|
|14|squeue|报告作业或作业步骤的状态。它有各种各样的筛选、排序和格式化选项。默认情况下，它按优先级顺序报告正在运行的作业，然后按优先级顺序报告挂起的作业。|
|15|sreport|根据 slurm 会计数据生成报告。|
|16|srun|用于提交要执行的作业或实时启动作业步骤。srun 有各种各样的选项来指定资源需求，包括: 最小和最大节点数、处理器数、要使用或不使用的特定节点，以及特定节点特性(多少的内存、磁盘空间、某些必需特性等)。一个作业可以包含多个作业步骤，这些作业步骤在作业节点分配的独立或共享资源上顺序或并行执行。|
|17|sshare|显示集群上公平共享使用关于的详细信息。请注意，这只有在使用优先级/多因素插件时才可行。|
|18|sstat|用于获取正在运行的作业或作业步骤所使用的资源的关于。|
|19|strigger|用于设置、获取或查看事件触发器。事件触发器包括节点下线或接近时间限制的作业等情况。|
|20|sview|是一个图形用户界面，用于获取和更新由 slurm 管理的作业、分区和节点的状态信息。|

## 1. sacct

### 用于报告关于活动或已完成的工作或工作步骤的计费信息。

查询已经完成的作业详情

```sh
[root@mn1 .sync-task]# sacct -j 3667
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
3667         pod-sync-+     normal       root          1  COMPLETED      0:0 
3667.batch        batch                  root          1  COMPLETED      0:0 
3667.extern      extern                  root          1  COMPLETED      0:0 
3667.0       sync-reso+                  root          1  COMPLETED      0:0 
```

## 2.sacctmgr

查询用户并且列出关联关系

```sh
[root@mn1 slurm]# sacctmgr show user withassoc
      User   Def Acct     Admin    Cluster    Account  Partition     Share   Priority MaxJobs MaxNodes  MaxCPUs MaxSubmit     MaxWall  MaxCPUMins                  QOS   Def QOS 
---------- ---------- --------- ---------- ---------- ---------- --------- ---------- ------- -------- -------- --------- ----------- ----------- -------------------- --------- 
        ad         ad  Operator     fs_dev         ad                    1                                                                                      normal    normal 
        ad         ad  Operator     fs_dev         ad     normal         1                                                                                      normal    normal 
     admin      admin  Operator     fs_dev      admin     normal         1                                                                                       admin     admin 
     admin      admin  Operator     fs_dev      admin                    1                                                                                       admin     admin 
    c80060     c80060  Operator     fs_dev     c80060     normal         1                                                                                      c80060    c80060 
        ck         ck      None     fs_dev         ck     normal         1                                                                                      normal    normal 
        ct         ct  Operator     fs_dev         ct                    1                                                                                      normal    normal 
        ct         ct  Operator     fs_dev         ct     normal         1                                                                                      normal    normal 
       ct2        ct2  Operator     fs_dev        ct2                    1                                                                                    high,low      high 
       ct2        ct2  Operator     fs_dev        ct2     normal         1                                                                                    high,low      high 
   dyj2024    dyj2024  Operator     fs_dev    dyj2024     normal         1                                                                                     dyj2024   dyj2024 
      feng       feng  Operator     fs_dev       feng     normal         1                                                                                        feng      feng 
feng0feng+ feng0feng+  Operator     fs_dev feng0feng+     normal         1                                                                        feng0feng0feng0feng0 feng0fen+ 
    feng10     feng10  Operator     fs_dev     feng10     normal         1                                                                                      feng10    feng10 
  feng1111   feng1111  Operator     fs_dev   feng1111     normal         1                                                                                    feng1111  feng1111 
     feng2      feng2  Operator     fs_dev      feng2     normal         1                                                                                       feng2     feng2 
   feng222    feng222  Operator     fs_dev    feng222     normal         1                                                                                     feng222   feng222 
     feng3      feng3  Operator     fs_dev      feng3     normal         1                                                                                       feng3     feng3 
   feng666    feng666  Operator     fs_dev    feng666                    1                                                                                     feng666   feng666 
   feng666    feng666  Operator     fs_dev    feng666     normal         1                                                                                     feng666   feng666 
       fzk        fzk  Operator     fs_dev        fzk     normal         1                                                                                         fzk       fzk 
      fzk1       fzk1  Operator     fs_dev       fzk1                    1                                                                        high,low,normal,qos+      fzk1 

```

查询QOS详细信息

```
feng1@ln1:~ # sacctmgr show qos ziyuanjianding format=Name%50,GrpTRES%50
								Name 														GrpTRES
------------------------------------ --------------------------------------------------------------
				      ziyuanjianding                                               cpu=100,mem=500G
```



## 3.salloc

### 用于实时为作业分配资源。这通常用于分配资源和生成 shell。然后使用 shell 执行 srun 命令来启动并行任务。

## 8.scontrol

### Scontrol 命令可用于报告节点、分区、作业、作业步骤和配置等关于的更详细信息。系统管理员也可以使用它进行配置更改。

查询分区信息

```sh
[root@mn1 ~]# scontrol show partition
PartitionName=normal
   AllowGroups=ALL AllowAccounts=ALL AllowQos=ALL
   AllocNodes=ALL Default=NO QoS=N/A
   DefaultTime=5-00:00:00 DisableRootJobs=NO ExclusiveUser=NO GraceTime=0 Hidden=NO
   MaxNodes=3 MaxTime=5-00:00:00 MinNodes=0 LLN=NO MaxCPUsPerNode=UNLIMITED MaxCPUsPerSocket=UNLIMITED
   Nodes=cn[1-2]
   PriorityJobFactor=50 PriorityTier=50 RootOnly=NO ReqResv=NO OverSubscribe=NO
   OverTimeLimit=NONE PreemptMode=OFF
   State=UP TotalCPUs=22 TotalNodes=2 SelectTypeParameters=NONE
   JobDefaults=(null)
   DefMemPerCPU=500 MaxMemPerNode=UNLIMITED
   TRES=cpu=22,mem=47700M,node=2,billing=37,gres/gpu=2
   TRESBillingWeights=CPU=1.0,Mem=0.25G,gres/gpu=2.0

PartitionName=normal_master
   AllowGroups=ALL AllowAccounts=ALL AllowQos=ALL
   AllocNodes=ALL Default=YES QoS=N/A
   DefaultTime=5-00:00:00 DisableRootJobs=NO ExclusiveUser=NO GraceTime=0 Hidden=NO
   MaxNodes=3 MaxTime=5-00:00:00 MinNodes=0 LLN=NO MaxCPUsPerNode=UNLIMITED MaxCPUsPerSocket=UNLIMITED
   Nodes=mn1
   PriorityJobFactor=50 PriorityTier=50 RootOnly=NO ReqResv=NO OverSubscribe=NO
   OverTimeLimit=NONE PreemptMode=OFF
   State=UP TotalCPUs=6 TotalNodes=1 SelectTypeParameters=NONE
   JobDefaults=(null)
   DefMemPerCPU=500 MaxMemPerNode=UNLIMITED
   TRES=cpu=6,mem=15800M,node=1,billing=11,gres/gpu=1
   TRESBillingWeights=CPU=1.0,Mem=0.25G,gres/gpu=2.0
```

查询节点信息

```sh
[root@mn1 ~]# scontrol show node mn1
NodeName=mn1 Arch=x86_64 CoresPerSocket=1 
   CPUAlloc=6 CPUEfctv=6 CPUTot=6 CPULoad=22.10
   AvailableFeatures=(null)
   ActiveFeatures=(null)
   Gres=gpu:GTX1660Super:1,npu:Ascend910:4
   NodeAddr=mn1 NodeHostName=mn1 Version=23.02.4
   OS=Linux 3.10.0-1160.71.1.el7.x86_64 #1 SMP Tue Jun 28 15:37:28 UTC 2022 
   RealMemory=15800 AllocMem=4596 FreeMem=425 Sockets=3 Boards=1
   State=ALLOCATED ThreadsPerCore=2 TmpDisk=0 Weight=1 Owner=N/A MCS_label=N/A
   Partitions=normal_master 
   BootTime=2024-03-27T08:50:41 SlurmdStartTime=2024-03-27T08:52:08
   LastBusyTime=2024-03-27T11:19:26 ResumeAfterTime=None
   CfgTRES=cpu=6,mem=15800M,billing=11,gres/gpu=1
   AllocTRES=cpu=6,mem=4596M
   CapWatts=n/a
   CurrentWatts=0 AveWatts=0
   ExtSensorsJoules=n/s ExtSensorsWatts=0 ExtSensorsTemp=n/s
   Comment=string
```

给每个节点同步配置文件,重新加载配置文件，并更新分区状态。

```sh
scontrol reconfigure
```

生成token

```sh
[root@mn1 .sync-task]# scontrol token username=root lifespan=84000
SLURM_JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDkxNzYxNzksImlhdCI6MTcwOTA5MjE3OSwic3VuIjoicm9vdCJ9.gLzSzr2OJsMLkzMf0Wrz8N_GPtiFlPGEG44BS1nz0qU
```

查看节点状态信息

```shell
scontrol show nodes gpu1
```

更新节点状态

```shell
scontrol update NodeName=node01 State=DRAIN
```

查询正在运行的作业的详情

```sh
[root@mn1 .sync-task]# scontrol show job 3662
JobId=3662 JobName=test-fzk1-2024-02-28-38158
   UserId=fzk1(1559) GroupId=slurmuser(500) MCS_label=N/A
   Priority=100 Nice=0 Account=fzk1 QOS=fzk1
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=1 Reboot=0 ExitCode=0:0
   RunTime=01:11:40 TimeLimit=4-04:00:00 TimeMin=N/A
   SubmitTime=2024-02-28T10:35:59 EligibleTime=2024-02-28T10:35:59
   AccrueTime=2024-02-28T10:35:59
   StartTime=2024-02-28T10:35:59 EndTime=2024-03-03T14:35:59 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2024-02-28T10:35:59 Scheduler=Backfill
   Partition=normal AllocNode:Sid=192.168.1.143:1440
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=cn1
   BatchHost=cn1
   NumNodes=1 NumCPUs=1 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   ReqTRES=cpu=1,mem=2G,node=1,billing=1
   AllocTRES=cpu=1,mem=2G,node=1,billing=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=1 MinMemoryNode=2G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=(null)
   WorkDir=/var/hpc-root/fzk1
   Comment=normal:94f6fd738ece40ba924d6066758b6c9f 
   StdErr=/var/hpc-root/fzk1/fzk1_sbase_2024-02-28-38152.error.3662
   StdIn=/dev/null
   StdOut=/var/hpc-root/fzk1/fzk1_sbase_2024-02-28-38152.out.3662
   Power=
   TresPerNode=
```

将用户加入某个 partition

```sh
ssacctmgr add user name=<用户组名> account=<用户组名> partition=<partition名称>
sacctmgr add user name=root account=root partition=gpu
```

查看 slurm 的集群配置

```shell
scontrol show config  # 显示所有 Slurm 配置
scontrol show partition <分区名>  # 查看特定分区配置
scontrol show node <节点名>  # 查看特定节点配置
```



## 12. sinfo

使用 sinfo 查看节点 gres 资源

```sh
sinfo -N -o "%N %P %c %m %G %T"  # 节点、分区、CPU数、内存、GPU、状态
sinfo -e  # 详细节点信息


(base) [root@mn1 ~]# sinfo -o "%20P %5c %10m %15f %10G %25l %N"
PARTITION            CPUS  MEMORY     AVAIL_FEATURES  GRES       TIMELIMIT                 NODELIST
normal               6+    15800+     (null)          gpu:GTX106 infinite                  cn[1-2]
normal_master*       6     15800      (null)          gpu:GTX166 5-00:00:00                mn1
(base) [root@mn1 ~]# sinfo -o "%N %G"
NODELIST GRES
cn[1-2] gpu:GTX1060:1,npu:Ascend910:4
mn1 gpu:GTX1660Super:1,npu:Ascend910:4,v_npu:v_Ascend910:60
```

## 14.squeue

查询作业队列

```sh
[root@mn1 .sync-task]# squeue 
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
              3662    normal test-fzk     fzk1  R    1:07:52      1 cn1
```

## 查看slurm版本

```sh
[root@slurmdbd-7cd68ddbb-5kt5k /]# slurmctld -V
slurm 23.02.4
[root@slurmdbd-7cd68ddbb-5kt5k /]# 
```

---

### 关联笔记
- [[slurm 守护进程]]
- [[slurm 配置]]
- [[slurm 用户费用统计查询]]
- [[slurm服务启动指令]]
