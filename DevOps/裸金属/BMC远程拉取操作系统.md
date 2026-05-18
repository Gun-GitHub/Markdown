[[裸金属]]
# 一、概述

带有 BMC（Baseboard Management Controller）的服务器，可以在不开机进入操作系统的情况下，通过带外管理能力远程安装、部署、启动操作系统。

常见方案包括：
- PXE 网络启动
- Virtual Media（虚拟光驱挂载 ISO）
- Redfish API 远程部署
- IPMI 虚拟介质
- iKVM 远程控制
- HTTP/iPXE 无盘安装

常见 BMC：
- iDRAC（Dell）
- iLO（HPE）
- iBMC（Huawei）
- XClarity Controller（Lenovo）
- ASPEED AST2600 系列
- OpenBMC
# 二、BMC 的作用

BMC 本质是服务器上的一个独立小系统。

即使：
- 主机断电
- 操作系统崩溃
- 没有 SSH
- 没有显示器

仍然可以通过 BMC：
- 开关机
- 查看控制台
- 修改 BIOS
- 挂载 ISO
- PXE 启动
- 安装操作系统
- 重装系统

典型架构：

```text
运维机器
    ↓
BMC 管理网口
    ↓
服务器 BMC
    ↓
控制主机 BIOS / 电源 / 启动项
```

# 三、最常见方式：PXE 网络启动安装系统

## 1. 工作原理

服务器开机后：
```text
BIOS/UEFI
→ PXE DHCP
→ 获取 TFTP
→ 下载 BootLoader
→ 下载 Kernel/initrd
→ 安装系统
```

需要提供：
- DHCP
- TFTP
- HTTP/FTP/NFS
- Kickstart / Preseed

## 2. 典型流程
### 第一步：配置 PXE Server
安装：
```bash
apt install dnsmasq tftpd-hpa
```

或：
```bash
yum install dhcp-server tftp-server
```

### 第二步：准备系统镜像

例如 Ubuntu：
```text
ubuntu-24.04-live-server-amd64.iso
```
挂载：
```bash
mount -o loop ubuntu.iso /mnt/iso
```
通过 HTTP 提供：
```bash
python3 -m http.server 8080
```

### 第三步：配置 PXE 引导
示例：
```text
DEFAULT linux
LABEL linux
KERNEL vmlinuz
APPEND initrd=initrd.img ip=dhcp url=http://192.168.1.10/ubuntu.iso
```

### 第四步：通过 BMC 设置 PXE 启动
在 BMC 中：
```text
Boot Option
→ PXE
→ One Time Boot
```
然后远程开机。

# 四、Virtual Media（虚拟光驱挂载 ISO）

这是最简单、最通用的方式。
## 原理
BMC 模拟一个 USB/CDROM：
```text
远程 ISO
→ BMC
→ 服务器识别为本地光驱
```
服务器会认为：
```text
插入了一张系统安装光盘
```

## 使用流程

### 1. 登录 BMC Web UI
例如：
```text
https://<bmc-ip>
```

### 2. 打开 Remote Console
通常叫：
- iKVM
- Remote Console
- HTML5 Console

### 3. 挂载 ISO

菜单：
```text
Virtual Media
→ CD/DVD
→ Mount ISO
```

可选择：
- 本地 ISO
- HTTP ISO
- NFS ISO
- CIFS ISO

### 4. 设置从 Virtual CDROM 启动

```text
Boot Override
→ Virtual CDROM
```

---

### 5. 重启服务器

服务器就会进入安装系统。

---

# 五、Redfish API 自动化部署

现代服务器更推荐使用 Redfish。

Redfish 是标准化 REST API。

---

## 查询电源状态
```bash
curl -k -u admin:password \
https://<bmc-ip>/redfish/v1/Systems/1
```

## 开机
```bash
curl -k -u admin:password \
-X POST \
-H "Content-Type: application/json" \
-d '{"ResetType":"On"}' \
https://<bmc-ip>/redfish/v1/Systems/1/Actions/ComputerSystem.Reset
```

## 设置一次 PXE 启动
```bash
curl -k -u admin:password \
-X PATCH \
-H "Content-Type: application/json" \
-d '{
  "Boot": {
    "BootSourceOverrideEnabled": "Once",
    "BootSourceOverrideTarget": "Pxe"
  }
}' \
https://<bmc-ip>/redfish/v1/Systems/1
```

## 挂载远程 ISO
不同厂商 API 不同。
例如部分 OpenBMC：
```bash
curl -k -u admin:password \
-X POST \
-H "Content-Type: application/json" \
-d '{
  "Image": "http://192.168.1.10/os.iso",
  "Inserted": true,
  "WriteProtected": true
}' \
https://<bmc-ip>/redfish/v1/Managers/1/VirtualMedia/CD/Actions/VirtualMedia.InsertMedia
```

# 六、IPMI 方式
较老服务器常用。
安装工具：
```bash
apt install ipmitool
```

## 查看电源状态
```bash
ipmitool -I lanplus -H <bmc-ip> -U admin -P password power status
```

## 开机
```bash
ipmitool -I lanplus -H <bmc-ip> -U admin -P password power on
```

## 重启
```bash
ipmitool -I lanplus -H <bmc-ip> -U admin -P password power reset
```

## 设置 PXE 启动
```bash
ipmitool chassis bootdev pxe
```

# 七、iPXE + HTTP 无盘安装（推荐大规模部署）
传统 PXE 有问题：
- TFTP 很慢
- 文件小
- 不适合大规模

现代方案：
```text
iPXE + HTTP
```

服务器：
```text
PXE
→ iPXE
→ HTTP 拉取 kernel/initrd
→ 自动安装
```

优点：
- 快
- 易扩展
- 支持脚本
- 支持动态配置

## iPXE 示例
```text
#!ipxe
kernel http://192.168.1.10/vmlinuz
initrd http://192.168.1.10/initrd.img
boot
```

# 八、自动化无人值守安装
## Ubuntu：Autoinstall
cloud-init：
```yaml
#cloud-config
autoinstall:
  version: 1
  identity:
    hostname: worker01
    username: ubuntu
    password: "$6$xxxx"
```

## CentOS/RHEL：Kickstart
```text
url --url=http://192.168.1.10/rocky
rootpw password
reboot
```

# 九、实际生产推荐方案

## 小规模

推荐：

```text
BMC Virtual Media + ISO
```

优点：

- 简单
- 稳定
- 通用

---

## 中大型集群

推荐：
```text
iPXE + HTTP + 自动化安装
```

配合：
- Redfish
- Ansible
- MAAS
- Foreman
- Cobbler
- OpenStack Ironic

# 十、OpenStack Ironic（裸金属管理）
Ironic 本质：
```text
OpenStack Bare Metal Provisioning
```

可以：

- 自动 PXE
- 自动装机
- 自动清盘
- 自动上下电
- 自动部署镜像

底层大量使用：

- IPMI
- Redfish
- PXE
- iPXE

非常适合：

- AI 集群
- HPC
- 裸金属云

---

# 十一、安全建议

BMC 风险很高。

建议：

- 独立管理网络
- 不暴露公网
- 修改默认密码
- 开启 HTTPS
- 禁止弱密码
- 限制 IP
- 定期升级 BMC Firmware

因为：

```text
拿到 BMC ≈ 拿到整台服务器
```

---

# 十二、总结

带 BMC 的服务器远程安装操作系统，本质依赖：

```text
BMC
+ 网络启动
+ 远程介质
+ 自动化安装
```

常见方案对比：

| 方案 | 适合场景 | 优点 | 缺点 |
|---|---|---|---|
| Virtual Media | 单机运维 | 简单 | 手工操作较多 |
| PXE | 批量部署 | 自动化 | 配置复杂 |
| iPXE + HTTP | 大规模集群 | 快速稳定 | 需要基础设施 |
| Redfish | 自动化平台 | 标准化 | 厂商兼容问题 |
| IPMI | 老设备 | 通用 | 功能较老 |

现代裸金属平台通常会组合：

```text
Redfish + iPXE + 自动化安装
```

形成完整裸金属自动化部署体系。

