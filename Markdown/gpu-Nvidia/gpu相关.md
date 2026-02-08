CUDA

  ## 安装

- Ubuntu版
  1. 安装GPU驱动

```sh
# 安装相关依赖
sudo apt-get install gcc-7 make -y
# 安装驱动
sudo apt-get update
sudo apt-get install ubuntu-drivers-common
sudo ubuntu-drivers autoinstall # 安装后需要重启系统
# 检查驱动版本
# 这个命令输出结果会展示驱动版本和cuda版本，但只有driver版本是真的，
# 显示的cuda版本是当前显卡支持的最高版本，cuda的真正版本要等装完cuda后使用nvcc -V查看
nvidia-smi 

```

```
2. 安装CUDA
```

根据nvidia-smi的输出，查看支持的最高cuda版本，选择合适的cuda版本。
到[nvidia官网](https://developer.nvidia.com/cuda-toolkit-archive)下载对应cuda版本的run文件，如安装CUDA 11.1:

```sh
wget https://developer.download.nvidia.com/compute/cuda/11.1.1/local_installers/cuda_11.1.1_455.32.00_linux.run
sudo sh cuda_11.1.1_455.32.00_linux.run --silent --toolkit
# 安装完之后需要将cuda路径添加到环境变量里面去
echo -e "export PATH=/usr/local/cuda-11.1/bin:\$PATH" >> ~/.bashrc
echo -e "export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc
# 检查安装是否成功
nvcc -V
```

```
3. 安装cudnn
```

cudnn同样需要到nvidia官网下载，要注意版本与cuda对应，下载完之后会是个压缩包，将压缩包解压，将其中的关键文件拷贝到cuda安装路径下即可。

```sh
#假设下载的压缩包文件名是cudnn.tgz
tar -zxvf cudnn.tgz 
sudo cp cuda/include/cudnn*.h /usr/local/cuda-11.6/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda-11.6/lib64
sudo chmod a+r /usr/local/cuda-11.6/include/cudnn*.h /usr/local/cuda-11.6/lib64/libcudnn*

```

```
    用户也可以考虑使用conda等虚拟环境来安装cuda和cudnn，这样可以实现多个不同的cuda版本环境，不同用户之间的环境可以相互隔离。
```

而且conda可以直接安装cuda-toolkit

- Debian版
  1. 下载并安装驱动
先使用lspci |grep VGA查看显卡型号，然后到nvidia官网上找对显卡型号对应版本的驱动并下载
  2. 禁用nouveau
     
      nouveau是通过逆向“Nvidia的Linux驱动”创造的一个开源第三方Nvidia显卡驱动程序，因此其效果差，性能低。**在手动安装NVIDIA时需要禁用nouveau驱动。**
     1. 终端执行以下命令修改文件。
        
         sudo vi /etc/modprobe.d/blacklist.conf
        
         以下内容复制到文件中
        
           blacklist nouveau   
        
           blacklist lbm-nouveau   
        
           options nouveau modeset=0 
        
           alias nouveau off   
        
           alias lbm-nouveau off
        
         补充说明：文件名其实不重要，重要的是文件内容和文件后缀用conf。
        
        b. 由于nouveau是构建在内核中的，所以要执行下面命令生效:
        
        sudo update-initramfs -u
        
        c. 重启
        
        reboot
        
        重启后查看nouveau有没有运行，没输出代表禁用生效
        
        lsmod |grep nouveau
  3. 关闭图形界面
     
      安装Nvidia驱动程序时，需要停止当前的图形界面。
     
      使用快捷键**CTRL+ALT+F2**进入超级终端，**登录账号**，并关闭图形界面：

（或者有时候使用快捷键失效，可以使用命令`sudo chvt 2`）

```
      sudo service lightdm stop

    说明：登录账号的时候，数字小键盘可能会默认关闭（即使小键盘灯是亮的)导致输入密码错误，启动下小键盘
4. 卸载以前的nvidia驱动
```

sudo apt autoremove *nvidia*
    5. 给驱动文件添加执行权限
sudo chmod +x NVIDIA***.run
    6. 安装驱动
sudo bash NVIDIA***.run
    7. 检验安装是否成功(有时候需要手动挂载驱动`modprobe nvidia`）
nvidia-smi
    8. 重启
    9. cuda和cudnn安装，同ubuntu版一致

- Centos版
  1. 核定内核版本
     1. 查看内核版本 `uname -r`
     2. 安装内核源代码 `yum install -y "kernel-devel-uname-r==$(uname -r)"
     3. `cd /usr/src/kernels` 查看和uname -r得到的同名版本的文件夹是否存在，要完全同名
  2. 安装依赖
`yum install -y gcc dkms pciutils `
  3. 禁用nouveau
     
      nouveau是通过逆向“Nvidia的Linux驱动”创造的一个开源第三方Nvidia显卡驱动程序，因此其效果差，性能低。**在手动安装NVIDIA时需要禁用nouveau驱动。**
     1. 终端执行以下命令修改文件。
        
         sudo vi /lib/modprobe.d/dist-blacklist.conf
        
         以下内容复制到文件中

```text
blacklist nouveau   
blacklist lbm-nouveau   
options nouveau modeset=0 
```

```
      b. 重新建立initramfs image文件

      `mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak`

      `dracut /boot/initramfs-$(uname -r).img $(uname -r)`

      c. 重启

      reboot

      重启后查看nouveau有没有运行，没输出代表禁用生效

      lsmod |grep nouveau
4. 下载驱动
```

根据自己的显卡型号和需要安装的cuda版本选择对应的驱动，下载
    5. 安装驱动

```
    如果之前安装过驱动,可以用 **`./驱动脚本 --uninstall`**   进行驱动的卸载，然后记得**重启**

    如果没有安装驱动,执行命令:  **`./驱动脚本 --kernel-source-path=/usr/src/kernels/(kernel的版本)`**
```

示例`./NVIDIA-Linux-x86_64-418.226.00.run --kernel-source-path=/usr/src/kernels/3.10.0-1160.el7.x86_64`

  ## 深度学习开发框架安装

  `conda install pytorch==2.0.1 torchvision==0.15.2 torchaudio==2.0.2 pytorch-cuda=11.6 -c pytorch -c nvidia`

  注意cuda版本要与上面安装的cuda版本对应

  ### 查看硬件的相关指令

  ls -l /dev/nvidia*
dmesg | grep -i nvidia
lspci | grep -i vga

  ## 安装中可能遇到的问题

  ### lspci能识别到卡但nvidia-smi无法识别到

```
原因有很多种可能，可参见下方多卡安装显卡识别不到的原因，但是额外要注意的是显卡的内存分配问题（尤其是华为的服务器），要排除此类问题需要进bios里面设置，不同bios可能还有差异，实在找不到设置的地方直接寻求官方文档

以华为purley平台为例，安装两张A100显卡后，只能识别其中一张。lspci命令能看到两张显卡，但nvidia-smi只能看到一张。

进一步使用lspci -vv |grep -C 20  "显卡对应的PCI端口"，可以看到显卡内存分配情况

![](https://secure2.wostatic.cn/static/ekr6HwzCcn3uh5ZrrckiyG/1d587f0f9901f5ed936ea35aceb9d0f.jpg?auth_key=1708398330-oX2H4iiYg6a8ZFoxqL3MNW-0-8cd4aaf0ded1c8e0ce7954fdfe86beec)

![](https://secure2.wostatic.cn/static/8vhVjEQKjNV8mW1igFjRk1/04dba6f4cc4df198a8e369900202881.jpg?auth_key=1708398329-wxctQcBNjxyQMfhfnMfMh4-0-217dfde2f0730493da406579d5b25378)

如上图所示一张卡分配了64G的内存，这张卡可以正常识别。另一张卡则有两个缓冲区没有未分配内存。推测原因是缓存区大小不够，给第一张卡分配完了之后第二张卡没有了。后向服务器官方客服寻求帮助，告知需要增大MMIO High Granularity Size（内存映射I/O高位粒度大小）。进入bios找到该项，发现默认设置为64G，也就是图上第一张卡分配掉的大小，改为256G重启后可正确识别两张卡。
```

[https://support.xfusion.com/support/#/zh/docOnline/EDOC1000163371?path=zh-cn_topic_0000001135898569&relationId=EDOC1000163372&mark=141](https://support.xfusion.com/support/#/zh/docOnline/EDOC1000163371?path=zh-cn_topic_0000001135898569&relationId=EDOC1000163372&mark=141)

```
**情况二**：有时候也会存在显卡分配了内存但仍然无法识别的情况，这种情况有可能是分配的内存太小，这个时候需要调整bios设置Enable above 4G PCI BAR 和 Enable Resize PCI BAR，允许分配大的内存映射空间。
```

  ## 多卡安装

  多显卡安装时，会出现有显卡识别不到的情况，可能的原因有：
    1. 硬件方面
        1. 电源功率不够，带不动多显卡
        2. 显卡连接不当/插错插槽
        3. 显卡本身损坏
    2. 软件方面
        1. bios设置问题
        2. pcie端口被禁用/显卡被禁用
        3. 驱动版本不对或驱动安装有问题

```
解决办法：
  1. xconfig添加gpu
```

```sh
# query the gpu information
nvidia-xconfig --query-gpu-info
# add the multiple gpu
nvidia-xconfig -a --device=Device0 --busid="PCI Bus ID of GPU #0" --device=Device1 --busid="PCI Bus ID of GPU #1"
reboot

```

```
  ![](https://secure2.wostatic.cn/static/qxhNTjrNWQUS5DChZiUESd/image.png?auth_key=1708398331-onXqud3nxmJs6vB8C1AHBs-0-cbe32e358b0d11c5565500e5b5ce7e52)

1. 取消显卡禁用

    检查`/sys/bus/pci/devices/0000:83:00.0/remove`文件，如果文件夹不存在，则表示设备未能正确启动，这种情况下建议检查电源供应是否足够，并确保GPU连接正确
```

如果文件中包含"1"，可以将其更改为"0"并重新启动以启用GPU。然而，如果文件夹存在但文件中没有包含值为"1"，则可能有其他因素导致故障。
    2. 蜜汁操作
        1. 修改/etc/modprobe.d/blacklist.conf，添加以下内容
blacklist nvidiafb
blacklist nouveau
        2. 执行命令

```
        `sudo depmod -ae`

        `sudo update-initramfs -u`

        ![](https://secure2.wostatic.cn/static/6b3qt7JrgpsyLgaMjU1VSG/image.png?auth_key=1708398329-vaon3iyRoaMbAnBVdq8E5J-0-c4da6aa70069f068fd6a9a24de2249ee)
```

  https://blog.csdn.net/qq_17783559/article/details/130928219#:~:text=%E5%8F%82%E8%80%83%E5%8D%9A%E5%AE%A2%EF%BC%9A%20(600%E6%9D%A1%E6%B6%88%E6%81%AF)%20ubuntu%2018.04%20%E4%B8%A4%E5%BC%A0GPU%E6%98%BE%E5%8D%A1%EF%BC%8Cnvidia-smi%E5%8F%AA%E6%98%BE%E7%A4%BA%E4%B8%80%E5%BC%A0_nvidia-smi%E5%8F%AA%E6%98%BE%E7%A4%BA%E4%B8%80%E5%BC%A0%E6%98%BE%E5%8D%A1_Jason.su.ai%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%201%E3%80%81%E4%BD%BF%E7%94%A8lspci%20%7Cgrep%20NVIDIA%E6%8C%87%E4%BB%A4%E7%9C%8B%E7%9C%8B%E6%98%BE%E5%8D%A1%E7%89%A9%E7%90%86%E8%BF%9E%E6%8E%A5%E6%98%AF%E5%90%A6%E5%87%BA%E7%8E%B0%E9%97%AE%E9%A2%98,%252Fdev%252Fnvidia*%E6%9F%A5%E7%9C%8Bnvidia%E9%A9%B1%E5%8A%A8%E6%98%AF%E5%90%A6%E6%AD%A3%E5%B8%B8%20%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B010%E5%9D%97%E6%98%BE%E5%8D%A1%E7%9A%84%E9%A9%B1%E5%8A%A8%E9%83%BD%E6%AD%A3%E5%B8%B8%E3%80%82%203%E3%80%81%E4%BD%BF%E7%94%A8echo%20%22hello%22%20%3E%20%252Fdev%252Fnvidia0%20%E6%9F%A5%E7%9C%8B%E9%80%9A%E4%BF%A1%E6%98%AF%E5%90%A6%E6%AD%A3%E5%B8%B8%20%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E7%AC%AC6%E5%9D%97%E6%98%BE%E5%8D%A1%E8%AF%BB%E5%86%99%E5%87%BA%E7%8E%B0%E9%94%99%E8%AF%AF%EF%BC%8C%E5%87%BA%E7%8E%B0%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E5%BA%94%E8%AF%A5%E5%B0%B1%E6%98%AF%E8%AF%A5%E5%9D%97%E6%98%BE%E5%8D%A1%E5%9D%8F%E6%8E%89%E4%BA%86%EF%BC%8C%E5%B0%91%E4%B8%80%E5%9D%97%E5%B0%B1%E5%B0%91%E4%B8%80%E5%9D%97%E5%90%A7%E3%80%82

  https://blog.csdn.net/yasugongyou1989/article/details/106534792/

  https://forums.developer.nvidia.com/t/only-one-of-two-rtx-3090s-detected-with-nvidia-smi-ubuntu-20-04/160103

  https://forums.developer.nvidia.com/t/new-installed-gpu-is-not-detected-by-nvidia-smi/165837
