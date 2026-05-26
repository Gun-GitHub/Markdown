## 📍 知识导航
- 🔗 [[0_裸金属知识体系|← 返回知识体系]]
- 📚 相关文档: [[BMC远程拉取操作系统]] | [[4_iPXE网络启动方案]]
- 👓 本文是对 [[4_iPXE网络启动方案|iPXE 方案]] 的技术参考

---

# IPXE 指令说明文档

## 概述

IPXE 是一个开源网络引导固件，提供了一套完整的指令用于网络引导、镜像加载和脚本执行。本文档列举了 IPXE 的主要指令及其使用方法。

---

## 目录

1. [网络相关指令](#网络相关指令)
2. [启动相关指令](#启动相关指令)
3. [镜像相关指令](#镜像相关指令)
4. [脚本控制指令](#脚本控制指令)
5. [菜单相关指令](#菜单相关指令)
6. [输入输出指令](#输入输出指令)
7. [系统控制指令](#系统控制指令)
8. [变量相关指令](#变量相关指令)
9. [实际应用示例](#实际应用示例)

---

## 网络相关指令

### dhcp
**功能**: 从 DHCP 服务器获取网络配置

```ipxe
dhcp
dhcp net0  # 指定网卡
```

**说明**:
- 自动获取 IP、网关、DNS 等网络配置
- 如果不指定网卡，则配置所有可用网卡
- 成功获取配置后可以使用网络相关功能

---

### ifopen / ifclose
**功能**: 打开或关闭网络接口

```ipxe
ifopen net0    # 打开网卡
ifclose net0   # 关闭网卡
```

**说明**:
- 用于控制特定网卡的状态
- 关闭后无法通过该网卡通信

---

### set
**功能**: 设置变量

```ipxe
set booturl http://mirror.example.com
set kernelpath /vmlinuz
set ${var_name} ${value}
```

**说明**:
- 可以设置任意名称的变量
- 变量使用 `${var_name}` 引用
- 支持嵌套变量替换

---

### unset
**功能**: 取消设置变量

```ipxe
unset booturl
unset ${var_name}
```

**说明**:
- 删除指定变量
- 变量被删除后无法再引用

---

## 启动相关指令

### boot
**功能**: 启动已加载的镜像

```ipxe
boot
boot kernel_name
```

**说明**:
- 执行之前通过 `imgload` 加载的镜像
- 不指定名称时启动最后加载的镜像
- 一旦执行 boot，控制权将转移给镜像

---

### chain
**功能**: 链式加载并执行另一个 IPXE 脚本或镜像

```ipxe
chain http://server/boot.ipxe
chain tftp://server/pxelinux.0
chain ${booturl}/boot.ipxe
```

**说明**:
- 支持 HTTP、HTTPS、TFTP、FTP 等协议
- 可以加载远程 IPXE 脚本并执行
- 支持变量替换

---

### sanboot
**功能**: SAN 启动（存储区域网络启动）

```ipxe
sanboot iscsi:server:3260:1:iqn.example.com:target
sanboot fc:wwn
```

**说明**:
- 用于从 iSCSI、光纤通道等存储设备启动
- 需要相应的存储协议支持

---

### exit
**功能**: 退出 IPXE 环境

```ipxe
exit
exit 0    # 成功退出
exit 1    # 异常退出
```

**说明**:
- 返回到 BIOS 或之前的启动程序
- 可以指定返回代码

---

## 镜像相关指令

### imgload
**功能**: 加载镜像文件到内存

```ipxe
imgload http://server/vmlinuz vmlinuz
imgload http://server/initrd.img initrd
imgload http://server/boot.iso
```

**说明**:
- 支持多种协议（HTTP、HTTPS、TFTP 等）
- 可以指定镜像名称，如未指定则使用 URL 的最后部分
- 镜像被加载到内存中，可供后续操作使用

---

### imgexec
**功能**: 执行已加载的镜像

```ipxe
imgexec
imgexec vmlinuz
imgexec vmlinuz initrd
```

**说明**:
- 执行指定的镜像
- 可以一次执行多个镜像
- 等同于 `boot` 命令

---

### imgfree
**功能**: 释放已加载的镜像

```ipxe
imgfree
imgfree vmlinuz
imgfree --all
```

**说明**:
- 删除指定的镜像
- 释放内存
- `--all` 选项删除所有镜像

---

### imgstat
**功能**: 显示已加载镜像的信息

```ipxe
imgstat
```

**说明**:
- 列出所有已加载的镜像
- 显示镜像名称、大小、类型等信息

---

## 脚本控制指令

### goto
**功能**: 无条件跳转到标签

```ipxe
goto end
:end
echo "Reached end"
```

**说明**:
- 跳转到指定的标签位置
- 用于实现条件分支和循环

---

### label
**功能**: 定义标签位置

```ipxe
:start
echo "Starting boot process"
:end
echo "Boot process complete"
```

**说明**:
- 定义脚本中的标签
- 可以与 `goto` 配合使用
- 标签名称以冒号开头

---

### iseq
**功能**: 相等条件判断

```ipxe
iseq ${bootmode} uefi || goto not-uefi
echo "UEFI mode"
:not-uefi
echo "Legacy mode"
```

**说明**:
- 比较两个字符串是否相等
- 相等则继续执行下一行
- 不相等则执行 `||` 后的命令
- 支持 `||`（或）和 `&&`（且）运算符

---

### isset
**功能**: 检查变量是否存在

```ipxe
isset ${booturl} || goto set-default
echo "Boot URL: ${booturl}"
:set-default
set booturl http://default.server
```

**说明**:
- 检查变量是否已设置
- 变量存在则继续
- 不存在则执行 `||` 后的命令

---

### sleep
**功能**: 延迟执行

```ipxe
sleep 5        # 延迟 5 秒
sleep 0.5      # 延迟 0.5 秒
```

**说明**:
- 暂停脚本执行指定时间
- 用于给用户反应时间或等待网络

---

## 菜单相关指令

### menu
**功能**: 定义菜单

```ipxe
menu Choose boot option
item ubuntu Ubuntu 20.04
item centos CentOS 8
item local Local disk
choose target && goto ${target}
```

**说明**:
- 定义启动菜单
- 后续的 `item` 定义菜单项
- `choose` 等待用户选择

---

### item
**功能**: 定义菜单项

```ipxe
item --gap           # 添加分隔符
item ubuntu Ubuntu 20.04
item centos CentOS 8
item --key 0 exit Exit menu
```

**说明**:
- 在菜单中添加项目
- `--gap` 添加分隔线
- `--key` 指定快捷键

---

### choose
**功能**: 显示菜单并等待用户选择

```ipxe
menu Boot Options
item local Local disk
item network Network boot
choose target
echo "You selected: ${target}"
```

**说明**:
- 显示菜单并等待选择
- 将选项的标签保存到指定变量
- 支持默认项和超时选项

---

## 输入输出指令

### echo
**功能**: 输出文本

```ipxe
echo "Booting system..."
echo "Server: ${server}"
echo
```

**说明**:
- 输出消息到控制台
- 支持变量替换
- 空 echo 输出空行

---

### read
**功能**: 从用户读取输入

```ipxe
read booturl
read --timeout 5 booturl http://default.server
echo "Boot URL: ${booturl}"
```

**说明**:
- 提示用户输入
- `--timeout` 指定超时秒数，超时使用默认值
- 输入值保存到指定变量

---

### show
**功能**: 显示变量值

```ipxe
show booturl
show all
```

**说明**:
- 显示指定变量的值
- `all` 显示所有变量

---

### clear
**功能**: 清屏

```ipxe
clear
```

**说明**:
- 清除屏幕上的所有内容
- 重置光标位置

---

## 系统控制指令

### reboot
**功能**: 重新启动计算机

```ipxe
reboot
```

**说明**:
- 执行系统重启
- 可用于启动失败后的恢复

---

### halt
**功能**: 停止系统执行

```ipxe
halt
```

**说明**:
- 停止脚本执行并等待用户干预
- 用于调试或等待手动操作

---

### cpuid
**功能**: 获取 CPU 信息

```ipxe
cpuid --ext 0x1 || goto no-sse
```

**说明**:
- 检查 CPU 功能支持
- 用于条件启动（如检查 UEFI 支持）

---

## 变量相关指令

### 内置变量

IPXE 提供以下常用内置变量：

| 变量 | 说明 |
|------|------|
| `${mac}` | MAC 地址 |
| `${mac:hexhyp}` | MAC 地址（十六进制-分隔） |
| `${ip}` | 当前 IP 地址 |
| `${gateway}` | 网关 IP |
| `${netmask}` | 子网掩码 |
| `${dns}` | DNS 服务器 |
| `${hostname}` | 主机名 |
| `${vendor}` | 固件厂商 |
| `${product}` | 产品名称 |
| `${serial}` | 序列号 |
| `${uuid}` | UUID |
| `${dhcp}` | DHCP 服务器地址 |
| `${filename}` | 启动文件名 |
| `${next-server}` | 下一个服务器 |
| `${boot-server}` | 启动服务器 |

**使用示例**:
```ipxe
echo "MAC: ${mac}"
echo "IP: ${ip}"
set bootfile http://server/${mac:hexhyp}.ipxe
```

---

## 实际应用示例

### 示例 1: 基本网络启动

```ipxe
# 获取 DHCP 配置
dhcp

# 输出启动信息
echo "Network boot starting..."
echo "MAC: ${mac}"
echo "IP: ${ip}"

# 加载内核和 initrd
imgload http://mirror.example.com/vmlinuz
imgload http://mirror.example.com/initrd.img

# 启动
boot
```

---

### 示例 2: 条件分支启动

```ipxe
dhcp

# 根据服务器型号选择不同的启动配置
iseq ${product} "Dell PowerEdge R640" && chain http://server/dell-r640.ipxe
iseq ${product} "HP ProLiant DL380 Gen10" && chain http://server/hp-dl380.ipxe

# 默认启动
chain http://server/default.ipxe
```

---

### 示例 3: 菜单选择启动

```ipxe
dhcp

# 定义菜单
menu Welcome to Boot Menu
item ubuntu Ubuntu 20.04
item centos CentOS 8
item local Local Disk
item shell IPXE Shell

# 等待用户选择
choose target && goto ${target}

# 各选项的处理
:ubuntu
echo "Booting Ubuntu..."
chain http://server/ubuntu/boot.ipxe
goto end

:centos
echo "Booting CentOS..."
chain http://server/centos/boot.ipxe
goto end

:local
echo "Booting from local disk..."
sanboot
goto end

:shell
echo "Entering IPXE shell..."
shell
goto end

:end
echo "Boot complete"
```

---

### 示例 4: 带超时的菜单和默认选项

```ipxe
dhcp

# 定义菜单
menu --timeout 10 Choose boot option (default: local)
item ubuntu Ubuntu 20.04
item --default local Local Disk

# 等待选择
choose --timeout 10 --default local target

# 处理选择
iseq ${target} ubuntu && chain http://server/ubuntu.ipxe || sanboot
```

---

### 示例 5: 使用变量和配置服务器

```ipxe
# 设置配置服务器
set configserver http://config.example.com

# 获取网络配置
dhcp

# 从配置服务器获取启动参数（返回 IPXE 脚本格式）
# 服务器返回: set booturl http://mirror.example.com
#           set kernelpath /vmlinuz
#           set initrdpath /initrd.img
chain ${configserver}/get-config.php?mac=${mac:hexhyp}&ip=${ip}

# 使用从服务器获取的变量
imgload ${booturl}${kernelpath}
imgload ${booturl}${initrdpath}

boot
```

---

### 示例 6: 错误处理和重试

```ipxe
dhcp

# 设置重试次数
set retry_count 0
set max_retries 3

:retry
iseq ${retry_count} ${max_retries} && goto max_retries_reached

echo "Attempting boot (attempt $((${retry_count}+1))/${max_retries})..."

# 尝试加载镜像
imgload http://mirror.example.com/vmlinuz || goto retry_failed
imgload http://mirror.example.com/initrd.img || goto retry_failed

boot

:retry_failed
set retry_count $((${retry_count}+1))
sleep 5
goto retry

:max_retries_reached
echo "Failed to boot after ${max_retries} attempts"
goto shell

:shell
shell
```

---

## 最佳实践

1. **总是先运行 dhcp**: 在执行任何网络操作前获取网络配置
2. **使用变量提高灵活性**: 将服务器地址、文件路径等设置为变量
3. **添加调试输出**: 使用 `echo` 输出关键步骤便于故障排查
4. **处理错误情况**: 考虑网络故障和超时，添加重试逻辑
5. **提供菜单选项**: 给用户选择启动模式的机会
6. **参数化配置**: 根据硬件特性（MAC、序列号等）选择不同配置
7. **测试充分**: 在生产环境前进行充分的测试

---

## 参考资源

- [IPXE 官方文档](https://ipxe.org/)
- [IPXE 命令参考](https://ipxe.org/cmd)
- [IPXE 脚本示例](https://ipxe.org/appnotes/buildandboot)

---

*文档更新时间: 2026-05-18*
