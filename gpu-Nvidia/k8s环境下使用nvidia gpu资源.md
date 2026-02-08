# k8s环境下使用nvidia gpu资源

```sh
1、第一步安装驱动
2、第二部安装docker runtime（只有在安装完dockerruntime之后k8s才能通过安装的插件来识别gpu）
3、k8s平台安装nvidia gpu插件
```

## 驱动下载

```sh
https://www.nvidia.cn/Download/index.aspx?lang=cn

这里选择红帽7的rpm包

#rpm包的安装步骤！！！（安装不成功，请改用run包！！！！！！）
rpm -i nvidia-driver-local-repo-rhel7-535.230.02-1.0-1.x86_64.rpm
yum clean all
yum install cuda-drivers
reboot

（这次没有安装，因为已经被安装好了）

检查驱动状态
nvidia-smi

查看相关软件包
rpm -qa | grep -i nvidia

查看驱动版本
 cat /proc/driver/nvidia/version
```

## 4090驱动安装（centos7）

```sh
下载之后需要处理部分错误才能安装成功
#先给个权限
chmod 777 NVIDIA-Linux-x86_64-550.78.run

#安装gcc和其他依赖
yum install gcc kernel-devel kernel-headers -y

配置此文件（这一部分必须配置，在重启之后即可开始安装）
vim /lib/modprobe.d/dist-blacklist.conf
注释此条
#blacklist nvidiafb
#追加两条
blacklist nouveau
options nouveau modeset=0

驱动安装
bash xxx.run

#提示X错误（说明系统还有如下某一种图形化视窗服务在运行，停止即可）
systemctl stop gdm
systemctl stop lightdm
systemctl stop sddm

#重建 initramfs image（驱动安装之后）（centos貌似不需要）
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)

#修改运行级别为文本模式
systemctl set-default multi-user.target

#重启
reboot

#检查（没有任何输出即可）
lsmod | grep nouveau


#后面那一节，需要看看是否有这个目录（开始安装）
./NVIDIA-Linux-x86_64-550.78.run --kernel-source-path /usr/src/kernels/3.10.0-1160.118.1.el7.x86_64


```

## 安装nvidia docker runtime

```
#驱动安装完成之后安装dockerruntime（官方指南）（这个指南是google找到的）
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/1.10.0/install-guide.html

docker自行安装完成之后，直接跳到文章中的 Setting up NVIDIA Container Toolkit 这一步，开始安装docker runtime（这里其实与gpt查询出来的答案一模一样）

1、添加yum存储库（直接执行）
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo

2、配置 /etc/docker/daemon.json 配置文件，设置默认的runtime为nvidia runtime ，示例如下
{
    ...
    "default-runtime": "nvidia",
    ...
}

===========================================（centos7.9新方法！！）
yum install -y nvidia-container-toolkit
#或者
dnf install -y nvidia-container-toolkit

#设置runtime（会提示成功）
nvidia-ctk runtime configure --runtime=docker

#检查（有输出即可）
nvidia-container-runtime -h

#（重启docker）
systemctl restart docker

#检查默认运行时
docker info | grep nvidia
#（看到输出如下配置即可）
# Default Runtime: nvidia
===========================================（ubuntu20.04新方法！！）
官网手册https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html  #可惜国内网络不可达

国内下载手动安装参考：https://blog.csdn.net/qq_33980935/article/details/140922446  只参考安装部分，因为博主用的是docker而不是containerd

国内可下载deb包的地址： https://mirror.cs.uchicago.edu/nvidia-docker/libnvidia-container/stable/ubuntu20.04/amd64/

#1、下载
wget https://mirror.cs.uchicago.edu/nvidia-docker/libnvidia-container/stable/ubuntu20.04/amd64/libnvidia-container1_1.13.5-1_amd64.deb
wget https://mirror.cs.uchicago.edu/nvidia-docker/libnvidia-container/stable/ubuntu20.04/amd64/libnvidia-container-tools_1.13.5-1_amd64.deb
wget https://mirror.cs.uchicago.edu/nvidia-docker/libnvidia-container/stable/ubuntu20.04/amd64/nvidia-container-runtime_3.13.0-1_all.deb
wget https://mirror.cs.uchicago.edu/nvidia-docker/libnvidia-container/stable/ubuntu20.04/amd64/nvidia-container-toolkit_1.13.5-1_amd64.deb
wget https://mirror.cs.uchicago.edu/nvidia-docker/libnvidia-container/stable/ubuntu20.04/amd64/nvidia-container-toolkit-base_1.13.5-1_amd64.deb

#2、安装
dpkg -i ./*.deb

#3、检查是否存在
whereis  nvidia-container-runtime 
which  nvidia-container-runtime 
====================================（下面是老方法，放弃使用）

3、安装runtime并重启
yum install -y nvidia-docker2
systemctl restart docker
（若docker版本没有问题，可以尝试使用dnf安装，chartGPT解释dnf本地环境的解析力更强）
yum install dnf
dnf install -y nvidia-docker2

4、 测试（能正常读取gpu信息则证明安装成功）
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi

```

## 安装k8s插件

```sh
1、下载 nvidia-device-plugin.yml 文件
#官方插件与教程地址，直接跳到Install the nvidia-container-toolkit步骤（有两个，推荐使用下面的连接下载nvidia-device-plugin.yml文件，下载时先找到最新的分支然后再下载）
#https://gitlab.com/nvidia/kubernetes/device-plugin
#https://github.com/NVIDIA/k8s-device-plugin/blob/master/nvidia-device-plugin.yml

#先下载到root目录（最新的版本需参照上面的方法和地址去查看）
curl https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.15.0/deployments/static/nvidia-device-plugin.yml > ~/nvidia-device-plugin.yml

#在yml文件中找到镜像并pull下来，私有化
docker pull nvcr.io/nvidia/k8s-device-plugin:v0.15.0
docker images
docker tag nvcr.io/nvidia/k8s-device-plugin:v0.15.0 idocker.io/nvidia/k8s-device-plugin:v0.15.0
docker push idocker.io/nvidia/k8s-device-plugin:v0.15.0

2、修改yml文件配置
#1、修改yml文件中的的镜像为本地镜像
#2、修改tag定位到gpu的节点
（tag说明）下载下来之后，指定运行的gpu服务器以及架构（指定服务器的原因是不一定所有的节点都带gpu）（指定架构的原因集群中有arm架构和x86架构，这个倒是不是所有项目都是这样）
在对应的节点上打上标签，以对应上述服务器条件
在每台x86 gpu节点上启动此插件

3、然后直接运行
kubectl create -f ./nvidia-device-plugin.yml

然后就可以在rancher、system、kube-system下看到nvidia开头的服务了

4、删除指定文件创建的pod（不需要操作）
kubectl delete -f nvidia-device-plugin.yml
```

4、问题解决

（没有设备被找到）

其他容器中使用gpu
docker run使用--gpu标志调用使您的硬件对容器可见，例如  --gpu=all
应该还能用于资源的分配，例如某个容器只分配部分gpu资源给其使用。
docker run -it --gpus all nvidia/cuda:11.4.0-base-ubuntu20.04 nvidia-smi
docker run --gpus="1" --name 容器名 -d -t 镜像id  （这里的单位貌似是一张显卡）
docker run --gpus all --name 容器名 -d -t 镜像id
