# 设置 WSL2 为默认版本

```powershell
wsl --set-default-version 2
```

# 安装Linux发行版

```powershell
# 列出可用的发行版
wsl --list --online

# 安装指定发行版（例如Ubuntu 22.04）
wsl --install -d Ubuntu-22.04

# 检查已安装的发行版及其WSL版本：
wsl --list --verbose

# 启动
wsl -d <发行版名称>

# 切换默认发行版
wsl --set-default <发行版名称>

# 重启WSL实例
wsl --shutdown

# 导出/导入发行版（备份或迁移）
wsl --export Ubuntu-22.04 ubuntu_backup.tar
wsl --import Ubuntu-22.04 C:\wsl\ubuntu C:\backup\ubuntu_backup.tar

# 卸载
wsl --unregister <发行版名称>

```

# 高级配置

修改WSL2配置文件
创建或编辑配置文件 %USERPROFILE%\.wslconfig，自定义资源分配：

```
[wsl2]
memory=4GB   # 限制内存使用
processors=2  # 分配CPU核心数
swap=2GB      # 交换空间大小
localhostForwarding=true
```

挂载Windows驱动器
WSL2默认将Windows驱动器挂载在 /mnt 目录（如 /mnt/c 对应 C:\）。
​注意：避免直接在Linux中修改Windows文件，可能引发权限问题。
​网络配置
WSL2使用虚拟化网络，其IP与主机不同。若需从Windows访问WSL2服务：
使用 localhost（如访问WSL2中的Web服务：http://localhost:8080）。
获取WSL2 IP：在Linux中运行 ip addr show eth0。
​启用GUI支持（Windows 11）​
Windows 11内置WSLg，支持直接运行Linux GUI应用。
安装GUI应用示例（如GEdit）：


```powershell
sudo apt install gedit -y
gedit

# 在 WSL 终端中运行，检查 WSLg 配置。若输出 :0 或类似值，表示 WSLg 已启用。
echo $DISPLAY

```

# 更改 wsl2 的存储位置 (以 docker，Ubuntu 为例)

## docker

1. 退出 docker

2. 查看 docker 挂载信息 wsl --list -v

3. 导出、注销、导入 docker-data, 其中需要建立对应文件夹 wsl\docker-data,wsl\docker,wsl\docker-
   ```powershell
   wsl --export docker-desktop-data "E:\wsl\wsl-data\docker-desktop-data.tar"
   wsl --unregister docker-desktop-data
   wsl --import docker-desktop-data "E:\wsl\docker\data" "E:\wsl\wsl-data\docker-desktop-data.tar" --version 2
   ```

4. 导出、注销、导入 docker-desktop
   ```powershell
   wsl --export docker-desktop "E:\wsl\wsl-data\docker-desktop.tar"
   wsl --unregister docker-desktop
   wsl --import docker-desktop "E:\wsl\docker\desktop" "E:\wsl\wsl-data\docker-desktop.tar" --version 2
   ```

<br/>

## ubunutu

新建 wsl\ubuntu 目录

```powershell
wsl --shutdown
wsl --export Ubuntu "E:\wsl\wsl-data\ubuntu.tar"
wsl --unregister Ubuntu
wsl --import Ubuntu "E:\wsl\ubuntu" "E:\wsl\wsl-data\ubuntu.tar" --version 2
#切换默认用户，我安装的时候设置的默认用户名是don
ubuntu config --default-user don
```