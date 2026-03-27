 # 为什么构建容器需要 Namespace

## 1.什么是 Namespace

Namespace 是 Linux 内核的一项功能，该功能对内核资源进行分区，以使一组进程看到组资源，而另一组进程看到另一组资源。Namespace 的工作方式通过为一组资源和进程设置相同的 Namespace 而起作用，但是这些 Namespace 引用了不同的资源。资源可能存在于多个 Namespace 中。这些资源可以是进程 ID、主机名、用户 |D、文件名、与网络访问相关的名称和进程间通信

最新的5.6版本的内核中实现了 8 中类型的 namespace:

|namesapce 名称|作用|内核版本|
|--|--|--|
|Mount(mnt)|隔离挂载点|2.4.19|
|Process ID(pid)|隔离进程 ID|2.6.24|
|Network(net)|隔离网络设备,端口号等|2.6.29|
|Interprocess Communication(ipc)|隔离 System V IPC 和 POSIX message queues(进程间通信机制)|2.6.19|
|UTS Namespace(uts)|隔离主机名和域名|2.6.19|
|User Namespace(user)|隔离用户和用户组|3.8|
|Control group(cgroup) Namespace|隔离 Cgroups 根目录|4.6|
|Time Namespace|隔离系统时间|5.6|

虽然最新操作系统内核中提供了 8 种 namespace, 但是 docker 只使用了其中的 6 种:

1. Mount(mnt):
   
   隔离不同的进程或者进程组看到的挂载点,实现容器内只能看到自己的挂载信息,在容器内的挂载操作不会影响主机的挂载目录,通俗的说就是,在不同的进程中看到不同的挂载目录
   
   unshare: 是 util-linux(centos7 已经默认已经继承该工具) 工具包中的一个工具,可以实现创建并访问不同类型的 Namespace
   
   ### 通过 `tmpfs` 来验证 `mount namespace`, `tmpfs`:
   
   `tmpfs` 是一种内存文件系统（Temporary File System），它将存储放在内存（RAM）中，而不是在磁盘上。这意味着 `tmpfs` 中的数据是暂时的，并且在系统重启后会丢失。`tmpfs` 主要用于那些需要快速访问和临时存储数据的情况。
   
   ##### 主要特点
   - **数据存储在内存中**：所有文件数据存储在系统内存中，存取速度快。
   - **临时文件存储**：`tmpfs` 中的数据仅在系统运行期间有效，当系统重启时，数据会丢失，因此适用于临时文件和缓存。
   - **动态大小调整**：`tmpfs` 会根据文件系统中数据的大小动态调整其所占用的内存空间，最多可占用全部可用内存的一部分（可以通过配置限制最大大小）。
   
   ##### 典型用途
   - 存放临时文件，例如 `/tmp` 或 `/run`。
   - 存放需要快速访问的缓存数据。
   - 防止频繁 I/O 影响硬盘寿命的应用场景。
   
   ##### 挂载示例
   
   可以使用以下命令挂载 `tmpfs`：
   ```sh
   mount -t tmpfs -o size=512M tmpfs /mnt/tmp
   ```
   
   在这个例子中，`tmpfs` 被挂载到 `/mnt/tmp`，大小限制为 512MB。
   
   ##### 优缺点
   - **优点**：速度快，因为数据存储在内存中；可以防止频繁的磁盘写操作。
   - **缺点**：内存容量有限，系统重启或挂载点卸载时数据会丢失。
   
   ### 验证Mount namespace:
   1. 创建一个 bash 进程并且新建一个 Mount Namespace
      ```sh
      $ sudo unshare --mount --fork /bin/bash
      # 返回信息
      [root@centos7 centos]#
      ```
   2. 在 /tmp 目录下创建一个目录
      ```sh
      [root@centos7 centos]# mkdir /tmp/tmpfs
      ```
   3. 使用 mount 命令挂载一个 tmpfs 类型的目录
      ```sh
      [root@centos7 centos]# mount -t tmpfs -o size=20m tmpfs /tmp/tmpfs
      ```
   4. 使用 df 命令查看已经挂载的信息
      ```sh
      [root@centos7 centos]# df -h
      Filesystem Size Used Avail Use% Mounted on
      /dev/vda1  500G 1.4G 499G  1%   /
      devtmpfs1  6G   0    16G   0%   /dev
      tmpfs      16G  0    16G   0%   /dev/shm
      tmpfs      16G  0    16G   0%   /sys/fs/cgroup
      tmpfs      16G  57M  16G   1%   /run
      tmpfs      3.2G 0    3.2G  0%   /run/user/1000
      tmpfs      20M  0    20M   0%   /tmp/tmpfs
      ```
   
          可以看到 /tmp/tmpfs 目录已经被正确挂载,为了验证主机目录并没有挂载此目录
   5. 新打开一个窗口, 执行 df -h 命令查看主机信息
      ```sh
      [centos@centos7 ~]$ df -h
      Filesystem Size Used Avail Use% Mounted on
      devtmpfs   16G  0   16G    0%   /dev
      tmpfs      16G  0   16G    0%   /dev/shm
      tmpfs      16G  57M 16G    1%   /run
      tmpfs      16G  0   16G    0%   /sys/fs/cgroup
      /dev/vda1  500G 1.4G 499G  1%   /
      tmpfs      3.2G 0   3.2G   0%   /run/user/1000
      ```
   
          主机上并没有看到 /tmp/tmpfs, 可见我们独立的 mount namespace 中执行 mount 操作,并不会影响主机的挂载
   6. 在当前的命令行窗口,查看当前的进程的 Namespace 信息
      ```sh
      [root@centos7 centos]# ls -l/proc/self/ns
      total 0
      lrwxrwxrwx.1 root root 0 Sep 4 08:20 ipc -> ipc:[4026531839]
      lrwxrwxrwx.l root root 0 Sep 4 08:20 mnt -> mnt:[4026532239]
      lrwxrwxrwx.1 root root 0 Sep 4 08:20 net -> net:[4026531956]
      lrwxrwxrwx.1 root root 0 Sep 4 08:20 pid -> pid:[4026531836]
      lrwxrwxrwx.1 root root 0 Sep 4 08:20 user -> user:[4026531837]
      lrwxrwxrwx.1 root root 0 Sep 4 08:20 uts -> uts:[4026531838]
      ```
   7. 新打开一个命令行窗口, 使用相同的命令查看主机上的 Namespace 信息
      ```sh
      [centos@centos7 ~]$ ls -l /proc/self/ns
      total 0
      lrwxrwxrwx.1 centos centos 0 Sep 4 08:20 ipc -> ipc:[4026531839]
      lrwxrwxrwx.1 centos centos 0 Sep 4 08:20 mnt -> mnt:[4026531840]
      lrwxrwxrwx.1 centos centos 0 Sep 4 08:20 net -> net:[4026531956]
      lrwxrwxrwx.1 centos centos 0 Sep 4 08:20 pid -> pid:[4026531836]
      lrwxrwxrwx.1 centos centos 0 Sep 4 08:20 user -> user:[4026531837]
      lrwxrwxrwx.1 centos centos 0 Sep 4 08:20 uts -> uts:[4026531838]
      ```
   
   	通过对比可以发现,除了 mount namespace 的 id 值不一样以外,其他 namespace ID 值均一样.
   
   通过以上结果我们可以得出结论,使用 unshare 命令可以新建 mount namespace , 并且在新建的 Mount namespace 内 mount 是和外部完全隔离的
2. Process ID(pid):
   
   PID namespace 的作用是用来隔离进程,在不同的 PID namespace 内,进程可以拥有相同的 PID 号. 利用 PID namespace 可以使得每个容器的主进程为 1 号进程.而容器中的进程在主机上却拥有不同的 PID.
   
   例如, 一个进程在主机上的 PID 为 122, 使用 PID namespace 可以实现该进程在容器内看到的 PID 为 1
   
   #### 验证 PID namespace
   1. 创建一个 bash 进程, 并且新建一个 PID namespace
      ```sh
      $ sudo unshare --pid --fork --mount-proc /bin/bash
      # 以下是输出
      [root@centos7 centos]#
      ```
   2. 在当前的命令行窗口使用 ps aux 命令查看进程信息
      ```sh
      [root@centos7 centos]# ps aux
      USER PID %CPU %MEM VSZ    RSS  TTY   STAT START TIME COMMAND
      root 1   0.0  0.0  115544 2004 pts/0 S    10:57 0:00 bash
      root 10  0.0  0.0  155444 1764 pts/0 R+   10:59 0:00 ps aux
      ```
3. Network(net):
   
   用来隔离网络设备, IP 地址 和 端口等信息
   
   Net Namespace 可以让每个进程拥有自己独立的 IP 地址, 端口 和 网卡信息,例如主机的 ip 地址为 172.16.4.1, 容器内可以设置独立的 IP 地址为 192.168.1.1
   
   #### 验证 Network Namespace
   1. 首先在宿主机上执行 ip a
      ```sh
      $ ip a
      1: lo: <LOOPBACK,UP,LOWER UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00inet 127.0.0.1/8 scope host lo
        valid lft forever preferred lft forever
        inet6 ::1/128 scope host
        valid lft forever preferred lft forever
      2: ethO: <BROADCAST,MULTICAST,UP,LOWER UP> mtu 1500 qdisc pfifo_ fast state UP group default qlen 1000
        link/ether 02:11:b0:14:01:0c brd ff:ff:ff:ff:ff:ff
        inet 172.20.1.11/24 brd 172.20.1.255 scope global dynamic eth0
        valid lft 86063337sec preferred lft 86063337secinet6 fe80::11:bOff:fe14:10c/64 scope link
        valid lft forever preferred lft forever
      3: docker0: <NO-CARRIER,BROADCAST.MULTICAST.UP> mtu 1500 qdisc noqueue state DoWN group defaultlink/ether 02:42:82:8d:a0:df brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.1/16 scope global docker0
        valid lft forever preferred lft forever
        inet6 fe80::42:82ff:fe8d:a0df/64 scope link
        valid lft forever preferred lft forever
      ```
   2. 使用以下命创建一个 net namespace
      ```sh
      $ sudo unshare --net --fork /bin/bash
      # 以下是输出
      [root@centos7 centos]
      ```
   3. 使用 ip a 命令查看一下网络信息
      ```
      [root@centos7 centos]# ip a
      1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
          link/ipip 0.0.0.0 brd 0.0.0.0
      ```
   4. 与主机网络设备信息不同
4. Interprocess Communication(ipc):
   
   主要是用来隔离进程间通信的
   
   PID namespace 和 IPC namespace 一起使用可以实现同一 IPC namespace 内的进程彼此可以通信,不同 IPC namespace 进程却不能通信
   ```sh
   ipcs -q 命令: 查看系统间通信队列列表
   ipcmk -Q 命令: 创建系统间通信队列
   ```
   
   #### 验证 IPC namespace
   1. 使用 unshare 命令创建一个 IPC Namespace
      ```sh
      $ sudo unshare --ipc --fork /bin/bash
      [root@centos7 centos]#
      ```
   2. 使用 ipcs -q 命令查看当前 IPC Namespace 下的系统通信队列列表
      ```sh
      [root@centos7 ~]$ ipcs -q
      
      ------ Message Queues --------
      key        msqid      owner      perms      used-bytes   messages    
      ```
   
       	由上可以看到当前系统无任何通信队列
   3. 使用 ipcmk -Q 命令创建一个系统通信队列
      ```sh
      [root@centos7 ~]$ ipcmk -Q
      Message queue id: 0
      ```
   4. 再次使用 ipcs -q 命令查看当前 IPC Namespace 下的系统通信队列列表
      ```sh
      [root@centos7 ~]$ ipcs -q
      
      ------ Message Queues --------
      key        msqid      owner      perms      used-bytes   messages    
      0xff13dc2b 0          root       644        0            0        
      ```
   
   		可以看到我们已经成功创建了一个系统通信队列
   5. 打开一个新的命令行窗口, 使用 ipcs -q 命令查看主机通信队列
      ```sh
      [centos@centos7 ~]$ ipcs -q
      
      ------ Message Queues --------
      key        msqid      owner      perms      used-bytes   messages  
      ```
   
   通过上面个的实验可以发现,在单独的 IPC Namespace 内创建的系统通信队列,在主机上无法看到, 即 IPC namespace 实现了系统通信队列的隔离
5. UTS Namespace(uts):
   
   主要是用来隔离主机名的,它允许每个 UTS Namespace 拥有一个独立的主机名
   
      #### 验证 UTS namesapce
   1. 使用 unshare 命令创建一个 UTS namespace
      ```sh
      $ sudo unshare --mount --fork /bin/bash
      # 以下是输出
      [root@centos7 centos]#
      ```
   2. 使用 hostname 命令 (hostname 用来查看主机名称) 设置主机
      ```
      [root@centos7 centos]# hostname -b mydocker
      ```
   3. 查看主机名
      ```
      [root@centos7 centos]# hostname
      mydocker
      ```
   4. 新打开一个窗口,使用相同的命令查看主机的 hostname
      ```
      [centos@centos7 ~]$ hostname
      centos7
      ```
6. User Namespace(user):
   
   主要是用来隔离用户和用户组的,使用 User Namespace 可以实现进程在容器内拥有 root 权限,而在主机上却是普通用户
User namespace 的创建可以不使用 root 权限
   
   #### 验证 User namespace
   1. 普通用户的身份创建一个 User namespace
      ```sh
      [centos@centos7 ~] $ unshare --user -r /bin/bash
      [root@centos7 ~]# 
      ```
   2. 执行 id 命令查看当前的用户信息
      ```sh
      [root@centos7 ~]# id
      uid=0(root) git=0(root) groups=0(root), 65534(nfsnobody)
      ```
   3. 在当前命令行窗口执行 reboot 指令
      ```sh
      [root@centos7 ~]# reboot
      Failed to open /dev/initctl: Permission denied
      Failed to talk to init daemon.
      ```

    可以看到,我们在新创建的 User namespace 内,虽然是 root 用户,但是并没有权限执行 reboot 命令,这说明在 User namespace 中并不能获得主机的 root 权限,也就是说 User namespace 实现了用户和用户组的隔离

## 2.为什么 Docker 需要 Namespace

Linux 内核从 2002年 2.4.19 版本开始加入了 Mount Namespace内核 3.8 版本加入了 User Namespace 为容器提供了足够的支持功能

当 Docker 新建一个容器时,会创建这六种 Namespace，然后将容器中的进程加入这些Namespace之中, 属于 docker 容器中的进程只能看见当前 Namespace 中的系统资源, 正是由于 docker 使用了Linux 中的这些 namespace 技术,才实现了namespace 的进程隔离, 可以说没有namespace 就没有 docker 容器

Namespace 是 Linux 内核的一个特性可以实现在同一主机系统中对进程ID、主机名、用户ID、文件名、网络和进程间通信等资源的隔离,正是由于 docker 使用了这些 namespace 技术,才实现了对容器进程的隔离