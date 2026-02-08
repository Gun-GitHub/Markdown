使用不同的 namespace , 可以实现容器的进程看不到别的容器资源

但是容器中的进程依然可以自由的使用主机的 cpu, memory 等相关资源,如果不加以限制,有可能会导致主机资源竞争,这就要使用到 linux 内核的另一个核心技术 cgroups

## 1. 什么是 cgroups

cgroups(全称:controlgroups)是Linux内核的一个功能可以实现限制进程或者进程组的资源(如 CPU、内存、磁盘IO 等)

cgroups 主要提供了以下功能:

1. 资源限制,限制资源的使用量
2.  优先级控制, 不同的组可以有不同的资源使用优先级
3. 审计, 计算控制组的资源使用情况
4. 控制, 控制进程的挂起或者回复

cgroup 实现依赖于三个核心概念:

1. 子系统(subsystem)
   
   是一个内核的组件一个子系统代表一类资源调度控制器
   
   例如,内存子系统可以限制内存的使用量,cpu 子系统可以限制 cpu 的使用时间
2. 控制组(cgroups)
   
   表示一组进程和一组带有参数的子系统的关联关系
   
   例如一个 进程使用了cpu子系统来限制cpu的使用时间,则这个进程和cpu子系统的关联关系称为控制组 
3. 层级树(hierarchy)
   
   是由一系列的控制组按照树状结构排列组成的子控制组默认拥有父控制组的属性
   
   比如系统中定义类控制组c1限制了cpu可以使用一核, 然后另外一个控制组c2 想实现限制cpu使用一核,同时限制使用内存 2g, 那么 c2 就可以直接继承 c1,无需重复定义 cpu 资源限制

cgroups 的 3 个核心概念中, 子系统是最核心的概念,因为子系统是真正实现某类资源限制的核心基础.

#### 在 linux 上默认启动的子系统

查看当前系统子系统信息:

```sh
$ sudo mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
```

这些子系统中 cpu 和 memory 子系统是容器环境中使用最多的子系统

## 2. 如何使用cgroups

#### 演示 cgroups 如何限制进程的 cpu 使用时间

一下命令使用 root 权限执行

1. cgroups 的创建很简单, 只需要在相应的子系统下创建目录即可
   ```
   mkdir /sys/fs/cgroup/cpu/mytest
   ```
2. 查看创建的文件夹即可看见文件夹下出现了很多文件
   ```sh
   # ls -al /sys/fs/cgroup/cpu/mytest/
   total 0
   drwxr-xr-x 2 root root 0 Oct 28 17:23 .
   drwxr-xr-x 8 root root 0 Oct 14 09:19 ..
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cgroup.clone_children
   --w--w--w- 1 root root 0 Oct 28 17:23 cgroup.event_control
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cgroup.procs
   -r--r--r-- 1 root root 0 Oct 28 17:23 cpuacct.stat
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cpuacct.usage
   -r--r--r-- 1 root root 0 Oct 28 17:23 cpuacct.usage_percpu
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cpu.cfs_period_us
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cpu.cfs_quota_us
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cpu.rt_period_us
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cpu.rt_runtime_us
   -rw-r--r-- 1 root root 0 Oct 28 17:23 cpu.shares
   -r--r--r-- 1 root root 0 Oct 28 17:23 cpu.stat
   -rw-r--r-- 1 root root 0 Oct 28 17:23 notify_on_release
   -rw-r--r-- 1 root root 0 Oct 28 17:23 tasks
   ```
   
   其中cpu.cfs_quarter_us 文件代表在某一个阶段限制的cpu时间总量,单位: ms. 例如,要限制某个进程最多使用 1 核 cpu, 就在这个文件写入10 万, task 文件写入进程的 id 即可,此时,我们所需要的 cgroup 就创建好了
3. 将 shell 进程加入 cgroups 中
   ```sh
   cd /sys/fs/cgroup/cpu/mytest
   echo $$ > tasks
   ```

4. 查看 tasks 文件内容
   ```sh
   cat tasks
   3485
   3543
   ```

其中第一进程 id 为当前 shell 的主进程

5. 制造一个死循环, 提升 cpu 使用率
   ```sh
   while ture;do echo;done;
   ```

6. 新建一个窗口, 使用 top -p 命令查看当前 cpu 使用率
   ```sh
   top -p 3485
   ```

		为了测试 cgroup 是否能够对进程资源做限制,我们修改 cpu 的限制时限为 0.5 核

7. 修改 cpu.cfs_quota_us 限制 shell 进程使用 cpu 的使用率
   ```sh
   echo 50000 > /sys/fs/cgroup/mytest/cpu.cfs_quota_us
   ```

		同样使用上面个的命令制造死循环查看 cpu 使用率

可以看到相同的,cpu 的使用率被限制为了 0.5 个核

#### 演示 cgroups 如何限制进程的 memory 使用大小

1. 在 memory 子系统下创建 cgroup
   ```sh
   mkdir /system/fs/cgroup/memory/mytest
   ```

2. 查看子系统下的目录可以看见以下内容
   ```sh
   # ls -al
    total 0                                                                              
   drwxr-xr-x  2 root root 0 Oct 28 21:08 .                                             
   dr-xr-xr-x 10 root root 0 Oct 28 20:47 ..                                            
   -rw-r--r--  1 root root 0 Oct 28 21:09 cgroup.clone_children                         
   --w--w--w-  1 root root 0 Oct 28 21:09 cgroup.event_control                          
   -rw-r--r--  1 root root 0 Oct 28 21:09 cgroup.procs                                  
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.failcnt                                
   --w-------  1 root root 0 Oct 28 21:09 memory.force_empty                            
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.kmem.failcnt                           
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.kmem.limit_in_bytes                    
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.kmem.max_usage_in_bytes                
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.kmem.tcp.failcnt                       
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.kmem.tcp.limit_in_bytes                
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.kmem.tcp.max_usage_in_bytes            
   -r--r--r--  1 root root 0 Oct 28 21:09 memory.kmem.tcp.usage_in_bytes                
   -r--r--r--  1 root root 0 Oct 28 21:09 memory.kmem.usage_in_bytes                    
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.limit_in_bytes                         
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.max_usage_in_bytes                     
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.memsw.failcnt                          
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.memsw.limit_in_bytes                   
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.memsw.max_usage_in_bytes               
   -r--r--r--  1 root root 0 Oct 28 21:09 memory.memsw.usage_in_bytes                   
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.move_charge_at_immigrate               
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.oom_control                            
   ----------  1 root root 0 Oct 28 21:09 memory.pressure_level                         
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.soft_limit_in_bytes                    
   -r--r--r--  1 root root 0 Oct 28 21:09 memory.stat                                   
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.swappiness                             
   -r--r--r--  1 root root 0 Oct 28 21:09 memory.usage_in_bytes                         
   -rw-r--r--  1 root root 0 Oct 28 21:09 memory.use_hierarchy                          
   -rw-r--r--  1 root root 0 Oct 28 21:09 notify_on_release                             
   -rw-r--r--  1 root root 0 Oct 28 21:09 tasks                                         
   ```

3. 对内存使用限制为 1G 
   ```sh
   cd /sys/fs/cgroup/memory/mytest
   echo 1073741824 > memory.limit_in_bytes
   ```

4. 将当前进程写入 task
   ```sh
   cd /sys/fs/cgroup/memory/mytest
   echo $$ > tasks
   ```

5. 使用内存测试工具测试内存
   ```sh
   memtester 1500M 1
   ```

		报错, got 1500MB ,trying mlock...killed

删除 cgroups, 只需要删除 对应的文件夹即可

```sh
rm -rf /sys/fs/cgroup/memory/mytest
rm -rf /sys/fs/cgroup/cpu/mytest
```

## 3. Docker 是如何使用 cgroups 的

创建一个 nginx 容器

```sh
docker run -it -m=1g nginx
```

使用 ls 命令查看 cgroups 内存子系统的目录下的内容

```sh
╭─root@Jun /mnt/c/Users/admin
╰─➤  ls -l /sys/fs/cgroup/memory
total 0
drwxr-xr-x  2 root root 0 Nov  2 12:42 001-metadata
drwxr-xr-x  2 root root 0 Nov  2 12:42 002-services0
drwxr-xr-x  2 root root 0 Nov  2 12:42 003-services1
drwxr-xr-x  2 root root 0 Nov  2 12:42 004-docker-net-root
drwxr-xr-x  2 root root 0 Nov  2 12:42 005-bridge
drwxr-xr-x  2 root root 0 Nov  2 12:42 01-dns-forwarder
drwxr-xr-x  2 root root 0 Nov  2 12:42 artifactory
drwxr-xr-x  2 root root 0 Nov  2 12:42 binfmt
-rw-r--r--  1 root root 0 Nov  2 12:42 cgroup.clone_children
--w--w--w-  1 root root 0 Nov  2 12:42 cgroup.event_control
-rw-r--r--  1 root root 0 Nov  2 12:41 cgroup.procs
-r--r--r--  1 root root 0 Nov  2 12:42 cgroup.sane_behavior
drwxr-xr-x  2 root root 0 Nov  2 12:42 container-filesystem
drwxr-xr-x  2 root root 0 Nov  2 12:41 dev-hugepages.mount
drwxr-xr-x  2 root root 0 Nov  2 12:41 dev-mqueue.mount
drwxr-xr-x  2 root root 0 Nov  2 12:42 devenv-service
drwxr-xr-x  4 root root 0 Nov  2 12:42 docker
drwxr-xr-x  2 root root 0 Nov  2 12:42 http-proxy
```

进入 docker 目录下查看相关内容:

```sh
(base) ╭─root@Jun /mnt/c/Users/admin
╰─➤  ls -l /sys/fs/cgroup/memory/docker
total 0
drwxr-xr-x 2 root root 0 Nov  2 12:42 9439f85e81c5bc45596427baf23e67ca637e541f8f37fdfbc72d7cea4e0b39b8
drwxr-xr-x 2 root root 0 Nov  2 17:16 a777072c0052545347021a2076806e5e2c020bb49bad021454122e7aba27a601
```

a777072c0052545347021a2076806e5e2c020bb49bad021454122e7aba27a601 目录即为我们这个 nginx 容器的 uuid

我们进入这个容器查看一目录内容

```sh
(base) ╭─root@Jun /sys/fs/cgroup/memory/docker/a777072c0052545347021a2076806e5e2c020bb49bad021454122e7aba27a601
╰─➤  ls
cgroup.clone_children           memory.kmem.tcp.max_usage_in_bytes  memory.oom_control
cgroup.event_control            memory.kmem.tcp.usage_in_bytes      memory.pressure_level
cgroup.procs                    memory.kmem.usage_in_bytes          memory.soft_limit_in_bytes
memory.failcnt                  memory.limit_in_bytes               memory.stat
memory.force_empty              memory.max_usage_in_bytes           memory.swappiness
memory.kmem.failcnt             memory.memsw.failcnt                memory.usage_in_bytes
memory.kmem.limit_in_bytes      memory.memsw.limit_in_bytes         memory.use_hierarchy
memory.kmem.max_usage_in_bytes  memory.memsw.max_usage_in_bytes     notify_on_release
memory.kmem.tcp.failcnt         memory.memsw.usage_in_bytes         tasks
memory.kmem.tcp.limit_in_bytes  memory.move_charge_at_immigrate
(base) ╭─root@Jun /sys/fs/cgroup/memory/docker/a777072c0052545347021a2076806e5e2c020bb49bad021454122e7aba27a601
╰─➤  cat memory.limit_in_bytes
1073741824
```

可以看到内存限制值正好为 1G

事实上 docker 创建容器时, docker 会根据启动容器的参数, 在对应的 cgroups 子系统下, 创建以容器 id 为名称的目录,然后根据容器启动时, 资源限制的参数,修改对应的 cgroups 子系统资源限制文件, 从而达到资源限制的效果.

注意: cgroups 虽然可以实现资源限制, 但是不能保证资源的使用

例如: cgroups 虽然能够限制某个容只能使用 1核 cpu,但不能保证总能使用到 1 核 cpu, 当 cpu 资源发生竞争时,可能会导致实际使用的 cpu 资源发生竞争
