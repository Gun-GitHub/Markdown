## 环境检查

确认是否具备quota工具库

```sh
[root@mn1 hpc-root]# rpm -qa |grep quota
quota-nls-4.01-19.el7.noarch
quota-4.01-19.el7.x86_64
[root@mn1 hpc-root]# rpm -qa |grep xfsprogs
xfsprogs-4.5.0-22.el7.x86_64
```

## 挂载盘时指定参数usrquota,grpquota

设置挂载点参数加入usrquota,grpquota 

```sh
vi /etc/fstab
UUID=27ef6e49-3c16-416f-962f-0da62a75e417 /data2                   xfs     defaults,usrquota,grpquota        0 0
```

解释:

  /etc/fstab:  这个文件是 Linux 中存储挂载点信息的文件，用于指定系统引导时需要挂载的文件系统。

  UUID=27ef6e49-3c16-416f-962f-0da62a75e417 /data2 xfs defaults,usrquota,grpquota 0 0：这是 /etc/fstab 文件中的一行。它描述了一个文件系统的挂载信息。具体来说：

- UUID=27ef6e49-3c16-416f-962f-0da62a75e417 是该文件系统的唯一标识符。
- /data2 是挂载点，即文件系统将被挂载到该目录下。
- xfs 表示要使用的文件系统类型。
- defaults,usrquota,grpquota 是挂载选项，这些选项指定了在挂载文件系统时应该启用的特性。在这里，usrquota 和 grpquota 表示启用用户和组的磁盘配额功能。
- 0 0 是用于文件系统检查的参数，这些参数通常设置为 0，表示不进行自动检查。

重新挂载此挂载点，使用mount |grep /data2 查询是否参数是否生效    

```sh
正确加载参数的输出是
[root@mn1 ~]# mount |grep data2
/dev/sda2 on /data2 type xfs (rw,relatime,attr2,inode64,usrquota,grpquota)


错误加载的输出是，此时建议重启机器
[root@mn1 /]# mount |grep /data2
/dev/sda2 on /data2 type xfs (rw,relatime,attr2,inode64,noquota)
```

1. /dev/sda2: 这是文件系统挂载的设备或分区的路径。通常，/dev/sda表示系统上的物理磁盘，数字部分（在这里是2）表示分区号。因此，/dev/sda2指的是第二个物理磁盘上的分区。
2. on /data2: 这表示文件系统被挂载到系统的 /data2 目录上。在 Linux 系统中，文件系统通常被挂载到目录树中的某个位置以便用户访问其内容。
3. type xfs: 这指明了文件系统的类型，这里是 XFS 文件系统。XFS 是一种高性能的日志文件系统，通常用于大型文件系统和高性能环境。
4. (rw,relatime,attr2,inode64,usrquota,grpquota): 这是一系列挂载选项，它们控制文件系统的行为和特性。具体来说：
   
    1). rw: 表示文件系统以读写模式挂载，即允许读取和写入操作。
   
    2).relatime: 表示文件系统启用相对访问时间更新。相对访问时间是指文件被访问的时间，但只有在其数据或元数据发生变化时才会更新。
   
    3).attr2: 表示文件系统启用了扩展属性（Extended Attributes）的支持，这允许文件和目录存储额外的元数据。
   
    4).inode64: 表示文件系统使用64位的inode编号。这允许文件系统支持更多的文件和更大的文件系统容。
   
    5).usrquota: 表示文件系统启用了用户级别的磁盘配额限制功能。这允许管理员限制每个用户的磁盘空间使用量。
   
    6).grpquota: 表示文件系统启用了组级别的磁盘配额限制功能。这允许管理员限制每个用户组的磁盘空间使用量。

## 配置nfs服务到硬盘分区的挂载点

### 实例:

nfs服务配置文件

```sh
vi /etc/exports
/data2/hpc-root 192.168.1.143/20(rw,sync,no_root_squash)
chmod 2770 /data2/hpc-root  
注：
在命令 chmod 2770 /data 中，每个数字代表了一组权限：
第一个数字 2：指定了特殊权限位 SUID （Set User ID），也称为设置用户 ID。当 SUID 位被设置在一个可执行文件上时，它使得在执行该文件时，该文件会以拥有者（owner）的权限而不是执行者（user）的权限来执行。在目录上设置 SUID 位的效果不明确，通常不会用到。

systemctl restart nfs  #启动nfs服务
```

### 解释:

/etc/exports 是用于配置 NFS（Network File System）服务器的文件。在Linux系统中，NFS服务器使用该文件来指定需要共享的目录及其访问权限。

具体来说，/etc/exports 文件中的每一行描述了一个共享目录的配置，包括目录路径、访问权限、以及允许访问该目录的主机或者网络段。每行的格式通常是：

```sh
<目录路径> <允许访问的主机或网络段>(<共享选项>)
```

例如：

```sh
/home/shared 192.168.1.0/24(rw,sync)
```

这行的意思是将 /home/shared 目录分享给位于 192.168.1.0/24 网络段内的主机，并赋予读写权限。其中 (rw,sync) 是共享选项，表示可读写并且同步写入。

在大多数Linux发行版中，NFS服务通常是作为默认安装的一部分。因此，你可以说Linux自带了NFS。然而，并不是所有的发行版都是这样，有些发行版可能需要额外的步骤来安装NFS服务器软件包。

如果Linux发行版没有默认安装NFS服务器，你可以通过以下步骤来安装NFS服务器：

	1.使用包管理器安装NFS服务器软件包：不同的Linux发行版使用不同的包管理器。你可以使用适合你的发行版的包管理器来安装NFS服务器软件包。例如：

		在基于Debian的系统（如Ubuntu）中，可以使用apt-get或apt来安装NFS服务器软件包：

```sh
sudo apt-get update
sudo apt-get install nfs-kernel-server
```

		在基于Red Hat的系统（如CentOS、Fedora）中，可以使用yum或dnf来安装NFS服务器软件包：

```
sudo yum install nfs-utils
```

	2.配置NFS服务器：安装完成后，你需要配置NFS服务器以指定共享目录、访问权限等信息。主要配置文件是 /etc/exports。你可以编辑该文件并添加共享目录的配置信息。

	3.重新加载NFS服务：完成配置后，重新加载NFS服务以使更改生效。你可以执行以下命令来重新加载NFS服务：

		在基于Debian的系统中：

```sh
sudo systemctl reload nfs-kernel-server
```

		在基于Red Hat的系统中：

```sh
sudo systemctl reload nfs
```

	4.启动NFS服务：如果NFS服务尚未运行，你可能需要启动NFS服务。你可以执行以下命令来启动NFS服务：

		在基于Debian的系统中：

```sh
sudo systemctl start nfs-kernel-server
```

		在基于Red Hat的系统中：

```sh
sudo systemctl start nfs
```

## 针对用户的quota配置与测试

### 实例:

在主nfs服务节点(mn1)物理环境进行quota配置，测试用户为zck，是环境中ldap用户

```sh
xfs_quota -x -c "limit -u bsoft=20M bhard=30M zck" /data2/   #zck在此实验环境为ldap用户  软限制20M 硬限制30M
mkdir -p /data2/hpc-root/zck && chown zck /data2/hpc-root/zck  #给用户创建目录 这里模拟家目录效果

xfs_quota -x -c “指令”
    -x ：专家模式：只有加了-x后面才能加c
    -c：后面加的就是指令，这个小节我们先来谈谈数据回报的指令
    指令：
    print：单纯的列出目前主机内的档案系统参数等数据
    df：与原本的df一样的功能，可以加上-b（block）-i（inode）-h（加上单位）等
    report：列出目前的quota项目，有-ugr（user/group/project）及-bi等数据
    state：说明目前支持quota的档案系统的信息，有没有起动相关项目等
```

原文链接：https://blog.csdn.net/ZripenYe/article/details/119103083

### 注：

软限制（soft limit）和硬限制（hard limit）是在 Unix/Linux 系统中用于限制资源使用的两种不同类型的限制。

1.**软限制（Soft Limit）**：

- 软限制是一个警告阈值，当达到或超过软限制时，系统会发送警告消息给用户或进程。软限制可以在运行时被修改。
- 当进程达到软限制时，它仍然可以继续使用资源，不会立即导致进程终止或失败。软限制主要用于提醒用户或进程它们的资源使用情况接近限制，可以采取适当的行动来调整资源使用。
- 用户或进程通常可以通过修改其资源使用模式或请求管理员来增加软限制。

2.硬限制（Hard Limit）：

- 硬限制是资源的实际最大限制，超过该限制将导致操作系统强制终止进程或拒绝资源分配请求。硬限制只能由超级用户（root）或系统管理员修改。
- 硬限制防止用户或进程无意中或恶意地占用过多的系统资源，以保护系统免受资源耗尽或拒绝服务攻击等问题的影响。
- 硬限制通常设置为软限制的较高值，以提供一定的缓冲区和保护。

查询配置和已使用额度

```sh
xfs_quota -x -c "report -ubih" /data2
```

在另外一节点(cn1)挂载nfs卷

```sh
mount mn1:/data2/hpc-root/ /mnt

#切换zck用户
cd /mnt/zck
su zck
```

### 测试1：未超过硬限制

```sh
dd if=/dev/zero of=out.log bs=1024k count=29  #写29MB大小的文件没有超过硬限制

[root@mn1 hpc-root]# xfs_quota -x -c "report -ubih" /data2   #在mn1上查询已用量
User quota on /data2 (/dev/sda2)
Blocks                            Inodes
User ID      Used   Soft   Hard Warn/Grace     Used   Soft   Hard Warn/Grace
root       481.3M      0      0  00 [------]     48      0      0  00 [------]
#277        18.7G      0      0  00 [------]   6.8k      0      0  00 [------]
quota1      27.7M    20M    30M  00 [6 days]      4      0      0  00 [------]
zck         27.7M    20M    30M  00 [6 days]      2      0      0  00 [------]
```

### 测试2： 超过硬限制

```sh
dd if=/dev/zero of=out.log bs=1024k count=31  #写31MB大小的文件超过硬限制
dd: closing output file ‘out.log’: Disk quota exceeded

[root@mn1 hpc-root]# xfs_quota -x -c "report -ubih" /data2 
User quota on /data2 (/dev/sda2)
                        Blocks                            Inodes              
User ID      Used   Soft   Hard Warn/Grace     Used   Soft   Hard Warn/Grace  
---------- --------------------------------- --------------------------------- 
root       481.3M      0      0  00 [------]     48      0      0  00 [------]
#277        18.9G      0      0  00 [------]   6.9k      0      0  00 [------]
quota1      27.7M    20M    30M  00 [6 days]      4      0      0  00 [------]
zck           30M    20M    30M  00 [6 days]      2      0      0  00 [------]
```

## 更新用户quota配额

更新配置采用再执行一遍直接覆盖的方式

```sh
xfs_quota -x -c "limit -u bsoft=20M bhard=35M zck" /data2/

[root@mn1 hpc-root]# xfs_quota -x -c "report -ubih" /data2 |grep zck   #配置更新了
zck           30M    20M    35M  00 [6 days]      2      0      0  00 [------]
```

清除用户的系统配额

```sh
#把指定用户的软限制（bsoft）和硬限制（bhard）都设置为 0，即不再限制该用户的磁盘配额。
xfs_quota -x -c 'limit bsoft=0 bhard=0 zck' /data2

[root@mn1 hpc-root]# xfs_quota -x -c "report -ubih" /data2 |grep zck
zck           35M      0      0  00 [------]      2      0      0  00 [------]

#测试3
[zck@cn1 zck]$ dd if=/dev/zero of=out.log bs=1024k count=36  #写31MB大小的文件超过硬限制
36+0 records in
36+0 records out
37748736 bytes (38 MB) copied, 0.691696 s, 54.6 MB/s
```

## 宽限期(默认7天)

在 XFS 文件系统中，quota 的 grace time 参数是指一个时间段，在这段时间内，用户或组的磁盘配额已经超出了限制，但是系统仍然允许超出配额的状态持续存在，而不会立即触发限制。这个时间段被称为“宽限期”（grace period）。

在宽限期内，用户或组可以超出其配额，但系统会记录这种情况，并在宽限期结束后开始执行限制。通常，这个宽限期允许用户有足够的时间来调整其使用的磁盘空间，以避免超出配额后立即被系统拒绝服务。

grace time 参数的单位通常默认是秒，您可以根据需要在 quota 配置中设置它。
