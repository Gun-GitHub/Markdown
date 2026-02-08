## 基础编译

安装依赖

```sh
apt-get install build-essential fakeroot devscripts equivs munge libmunge-dev
groupadd -g 200 slurm && useradd -u 200 -g 200 -s /sbin/nologin -M slurm
```

拷贝源码

```sh
scp 192.168.1.143:/root/slurm-23.02.4.tar.bz2 .
```

编译源码

```
tar -xaf slurm*tar.bz2
cd slurm-23.02.4/
./configure
make install
```

修改/创建 slurmd.service

```
root@cn1:~# vi /lib/systemd/system/slurmd.service
[Unit]
Description=Slurm node daemon
After=munge.service network-online.target remote-fs.target
Wants=network-online.target
#ConditionPathExists=/usr/local/etc/slurm.conf

[Service]
Type=simple
EnvironmentFile=-/etc/sysconfig/slurmd
EnvironmentFile=-/etc/default/slurmd
ExecStart=/usr/local/sbin/slurmd --conf-server slurmctld.slurm-cluster:6817 -D -s $SLURMD_OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
LimitNOFILE=131072
LimitMEMLOCK=infinity
LimitSTACK=infinity
Delegate=yes
TasksMax=infinity
Restart=always  
RestartSec=5s

# Uncomment the following lines to disable logging through journald.
# NOTE: It may be preferable to set these through an override file instead.
#StandardOutput=null
#StandardError=null

[Install]
WantedBy=multi-user.target
```

修改 /etc/hosts, 记得把 容器 slurmctld 改成主机网络模式或者 service 对外暴露端口号

```sh
192.168.1.47 slurmctld.slurm-cluster
```

将slurmcld中的/etc/munge/munge.key 弄到物理环境

```sh
scp /root/slurm/munge.key gpu2:/etc/munge/munge.key
chown munge:munge /etc/munge/munge.key
systemctl restart munge.service
```

修改slrumctld 配置文件 /etc/slurm/slurm.conf

```sh
# slurm.conf
#
# See the slurm.conf man page for more information.
#
ClusterName=fs_dev
ControlMachine=mn1
ControlAddr=mn1
SlurmUser=slurm
SlurmctldPort=6817
SlurmdPort=6818
AuthType=auth/munge
StateSaveLocation=/var/lib/slurmd
SlurmdSpoolDir=/var/spool/slurmd
SwitchType=switch/none
MpiDefault=none
SlurmctldPidFile=/var/run/slurmd/slurmctld.pid
SlurmdPidFile=/var/run/slurmd/slurmd.pid
ProctrackType=proctrack/cgroup

ReturnToService=0
SlurmctldTimeout=300
SlurmdTimeout=300
InactiveLimit=0
MinJobAge=300
KillWait=100
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
JobAcctGatherType=jobacct_gather/cgroup
JobAcctGatherFrequency=30
AccountingStorageType=accounting_storage/slurmdbd
AccountingStorageHost=mn1
AccountingStoragePort=6819
AccountingStorageEnforce=associations #使用其它帐户提交
AuthAltTypes=auth/jwt
#AuthAltParameters=jwt_key=/etc/slurm/jwt_hs256.key
AuthAltParameters=jwt_key=/etc/slurm/jwt.key

# srun 可使用的端口号区间
# SrunPortRange = 30001-30005

# 资源计费相关
PriorityType=priority/multifactor
PriorityWeightTRES=CPU=1000,Mem=2000,gres/gpu=3000
AccountingStorageTRES=gres/gpu,gres/gpu:GTX1060,gres/gpu:GTX1660Super,gres/npu:Ascend910,gres/v_npu:v_Ascend910#,gres/npu:Ascend310,

# remote get config
SlurmctldParameters=enable_configless

# COMPUTE NODES
GresTypes=gpu,npu,v_npu
NodeName=mn1 CPUs=6 RealMemory=15800 Sockets=1 CoresPerSocket=6  ThreadsPerCore=1 State=UNKNOWN Gres=gpu:GTX1660Super:1,npu:Ascend910:4,v_npu:v_Ascend910:60
NodeName=cn1 CPUs=16 RealMemory=31900 Sockets=1 CoresPerSocket=8  ThreadsPerCore=2 State=UNKNOWN Gres=gpu:GTX1060:1,npu:Ascend910:4
NodeName=cn2 CPUs=6 RealMemory=15800 Sockets=1 CoresPerSocket=6  ThreadsPerCore=1 State=UNKNOWN Gres=gpu:GTX1060:1,npu:Ascend910:4

# 比较新的方式
NodeName=cn1 NodeAddr=192.168.1.203 NodeHostName=cn1 CPUs=40 RealMemory=32082 Sockets=2 CoresPerSocket=20 ThreadsPerCore=1 State=IDLE


# Resources Management
TaskPlugin=task/cgroup,task/affinity
PrologFlags=Contain

#
# PARTITIONS
# PartitionName=normal TRESBillingWeights="CPU=1.0,Mem=0.25G,gres/gpu=2.0" Default=yes Nodes=mn1,cn[1-2] Priority=50 DefMemPerCPU=500 Shared=NO MaxNodes=3 MaxTime=5-00:00:00 DefaultTime=5-00:00:00 State=UP

# 设置不限时 2024-06-07
# PartitionName=normal TRESBillingWeights="CPU=1.0,Mem=0.25G,gres/gpu=2.0" Default=yes Nodes=cn[1-2] Priority=50 DefMemPerCPU=500 Shared=NO MaxNodes=3 MaxTime=5-00:00:00 DefaultTime=5-00:00:00 State=UP
PartitionName=normal TRESBillingWeights="CPU=1.0,Mem=0.25G,gres/gpu=2.0" Default=yes Nodes=cn[1-2] Priority=50 DefMemPerCPU=500 Shared=NO MaxNodes=2 DefaultTime=5-00:00:00 State=UP

PartitionName=normal_master TRESBillingWeights="CPU=1.0,Mem=0.25G,gres/gpu=2.0" Default=yes Nodes=mn1 Priority=50 DefMemPerCPU=500 Shared=NO MaxNodes=1 MaxTime=5-00:00:00 DefaultTime=5-00:00:00 State=UP
```

启动 slurmd

```sh
systemctl reload
systemctl enable slurmd
systemctl restart slurmd
```

## 遇到的问题

#### 1). slurm 启动不了:

```
sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
normal*      up   infinite      1   invid cn1

```

查看 slurmctld 资源

```sh
slurmctld: debug:  Node cn1 has low real_memory size (32082 / 32768) < 100.00%
slurmctld: error: Setting node cn1 state to INVAL with reason:Low RealMemory (reported:32082 < 100.00% of configured:32768)
slurmctld: drain_nodes: node cn1 state set to DRAIN
```

重新修改 cn1 的内存大小,然后:

```
systemctl restart slurmd
```

#### 2). slurmd无法启动

```sh
slurmd: error: Couldn't find the specified plugin name for auth/munge looking at all files
```

https://www.reddit.com/r/SLURM/comments/17jkor4/problem_with_finding_munge/

```sh
apt install libmunge-dev -y

root@gpu1:~/slurm/slurm-23.02.4# dpkg -l |grep munge
ii  libmunge-dev                               0.5.13-2build1                      amd64        authentication service for credential -- development package
ii  libmunge2                                  0.5.13-2build1                      amd64        authentication service for credential -- library package
ii  munge                                      0.5.13-2build1                      amd64        authentication service to create and validate credentials

重新编译slurm
./configure
make install
```

## 配置k8s dns

```sh
mkdir -p /etc/resolvconf/resolv.conf.d/
vim /etc/resolvconf/resolv.conf.d/head
...文件内容
nameserver 169.254.25.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
.......


vi /etc/systemd/resolved.conf
[Resolve]
#DNS=
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#DNSOverTLS=no
#Cache=no-negative
#DNSStubListener=yes
#ReadEtcHosts=yes
DNS=169.254.25.10

Domains=default.svc.cluster.local svc.cluster.local cluster.local

systemctl restart  systemd-resolved.service
resolvectl status
```