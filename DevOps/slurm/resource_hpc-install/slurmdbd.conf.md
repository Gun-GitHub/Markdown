```yaml
ClusterName=fs_dev
ControlMachine=slurmctld
ControlAddr=slurmctld
SlurmUser=slurm
#SlurmdUser=root
SlurmctldPort=6817
SlurmdPort=6818
AuthType=auth/munge
StateSaveLocation=/data/slurmd
SlurmdSpoolDir=/var/spool/slurmd
SwitchType=switch/none
MpiDefault=none
SlurmctldPidFile=/var/run/slurmd/slurmctld.pid
SlurmdPidFile=/var/run/slurm/slurmd.pid
#ProctrackType=proctrack/linuxproc
ProctrackType=proctrack/cgroup
ReturnToService=0
SlurmctldTimeout=300
SlurmdTimeout=300
InactiveLimit=0
MinJobAge=300
KillWait=30
Waittime=0
SchedulerType=sched/backfill
SelectType=select/cons_tres
SelectTypeParameters=CR_CPU_Memory
#FastSchedule=1
SlurmctldDebug=3
SlurmctldLogFile=/var/log/slurm/slurmctld.log
SlurmdDebug=3
SlurmdLogFile=/var/log/slurm/slurmd.log
JobCompType=jobcomp/filetxt
JobCompLoc=/var/log/slurm/jobcomp.log
#JobAcctGatherType=jobacct_gather/linux
#JobAcctGatherType=jobacct_gather/cgroup
JobAcctGatherFrequency=30
AccountingStorageType=accounting_storage/slurmdbd
AccountingStorageHost=slurmdbd
AccountingStoragePort=6819
AccountingStorageEnforce=associations,limits #使用其它帐户提交
AuthAltTypes=auth/jwt
AuthAltParameters=jwt_key=/etc/slurm/jwt_hs256.key
SlurmctldParameters=enable_configless


# 如果使用 cgroup v2，可能需要调整插件
#TaskPlugin=task/affinity
JobAcctGatherType=jobacct_gather/linux
#
# COMPUTE NODES                                                 #修改(若无gpu、npu资源则直接去掉后面的参数）

NodeName=cn[3-5] CPUs=8 RealMemory=18432 Sockets=1 CoresPerSocket=8 ThreadsPerCore=1 State=IDLE 
#NodeName=cpu2 CPUs=8 RealMemory=5120 Sockets=1 CoresPerSocket=8 ThreadsPerCore=1 State=IDLE 
#NodeName=cpu3 CPUs=8 RealMemory=5120 Sockets=1 CoresPerSocket=8 ThreadsPerCore=1 State=IDLE 
#NodeName=cn1 CPUs=8 RealMemory=7168 Sockets=1 CoresPerSocket=8 ThreadsPerCore=1 State=IDLE Gres=gpu:GTX1660Super:1,npu:Ascend910:4
#NodeName=cn2 CPUs=8 RealMemory=7168 Sockets=1 CoresPerSocket=8 ThreadsPerCore=1 State=IDLE Gres=gpu:GTX1050Ti:1,npu:Ascend910:4
#
# Resources Management
TaskPlugin=task/cgroup,task/affinity
PrologFlags=Contain
#
# PARTITIONS                                                   #修改(根据实际情况增减分区）
#PartitionName=arm Nodes=ALL Default=YES MaxTime=INFINITE State=UP
PartitionName=x86 Nodes=cn[3-5] Default=YES MaxTime=INFINITE State=UP
```