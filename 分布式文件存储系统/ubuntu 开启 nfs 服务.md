Ubuntu 上开启 NFS 服务，通常分为「服务端」和「客户端」两部分。
# 一、安装 NFS 服务端
Ubuntu 服务端安装：
```bash
sudo apt update
sudo apt install -y nfs-kernel-server
```

安装完成后：
```bash
systemctl status nfs-kernel-server
```

启动并设置开机自启：
```bash
sudo systemctl enable --now nfs-kernel-server
```

# 二、创建共享目录

例如共享：
```bash
sudo mkdir -p /data/share
```

修改权限（测试阶段可以先放宽）：
```bash
sudo chmod 777 /data/share
```

# 三、配置 NFS 导出目录

编辑：
```bash
sudo nano /etc/exports
```

添加：
```bash
/data/share *(rw,sync,no_subtree_check)
```

含义：

|参数|说明|
|---|---|
|rw|可读写|
|sync|同步写入|
|no_subtree_check|提高稳定性|
|*|允许所有主机访问|

更安全的写法：
```bash
/data/share 192.168.1.0/24(rw,sync,no_subtree_check)
```
只允许局域网访问。

# 四、使配置生效

```bash
sudo exportfs -rav
```
查看共享：
```bash
sudo exportfs -v
```

# 五、放行防火墙（如果启用了 ufw）
```bash
sudo ufw allow nfs
```

或者：
```bash
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

# 六、客户端挂载

客户端安装：
```bash
sudo apt install -y nfs-common
```

创建挂载点：
```bash
sudo mkdir -p /mnt/nfs
```

挂载：
```bash
sudo mount -t nfs 服务器IP:/data/share /mnt/nfs
```

例如：
```bash
sudo mount -t nfs 192.168.1.10:/data/share /mnt/nfs
```

查看是否成功：
```bash
df -h
```

# 七、开机自动挂载

编辑：
```bash
sudo nano /etc/fstab
```

添加：
```bash
192.168.1.10:/data/share /mnt/nfs nfs defaults,_netdev 0 0
```

测试：
```bash
sudo mount -a
```
没有报错即可。

# 八、常用排查命令

查看服务：
```bash
systemctl status nfs-kernel-server
```

查看监听：
```bash
ss -tulpn | grep nfs
```

查看导出：
```bash
showmount -e localhost
```

客户端查看：
```bash
showmount -e 服务器IP
```

# 九、常见问题

## 1. mount: access denied by server
通常是：
- `/etc/exports` 配置不对
- IP 不在允许范围
- 没执行：
```bash
sudo exportfs -rav
```

## 2. Permission denied
通常是目录权限问题：
```bash
chmod 777
```

或者：
```bash
chown nobody:nogroup
```

## 3. Ubuntu 22/24 使用 NFSv4
有些系统默认只开 v4：
查看：
```bash
cat /proc/fs/nfsd/versions
```

# 十、推荐配置（局域网最常用）

服务端 `/etc/exports`：
```bash
/data/share 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

应用：
```bash
sudo exportfs -rav
```

客户端：
```bash
sudo mount -t nfs 192.168.1.10:/data/share /mnt/nfs
```