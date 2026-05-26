## 1. IPMI 架构与工作原理

### 核心组件
```
┌─────────────────────────────────────────┐
│         操作系统 / 应用                  │
├─────────────────────────────────────────┤
│    IPMI 客户端（ipmitool 等工具）       │
├─────────────────────────────────────────┤
│  IPMI 通信层（LAN、串口、IMPI消息总线）  │
├─────────────────────────────────────────┤
│     BMC（Baseboard Management Controller）│
│  ├─ 传感器模块                          │
│  ├─ 日志模块（SEL）                     │
│  ├─ 电源管理                            │
│  ├─ 远程访问（Serial Over LAN）         │
│  └─ BIOS/固件控制                       │
└─────────────────────────────────────────┘
```

### 三种通信模式
1. **In-band**：通过操作系统与 BMC 通信（系统运行时）
2. **Out-of-band**：直接网络连接 BMC（系统关闭也能用）
3. **Serial**：通过串口连接

---

## 2. IPMI 命令体系

### 核心 IPMI 命令分类

| 命令类型 | 功能 | 示例 |
|---------|------|------|
| **Power Control** | 电源管理 | on/off/reset |
| **Chassis** | 机箱状态 | 获取 LED、风扇状态 |
| **Sensor** | 硬件监控 | 温度、电压、风扇 |
| **SEL** | 系统事件日志 | 查看硬件错误历史 |
| **SDR** | 传感器数据记录 | 传感器列表 |
| **User** | 用户管理 | 创建/删除用户 |
| **Session** | 会话管理 | 远程连接管理 |

---

## 3. 实际使用 - ipmitool 工具

### 安装
```bash
# Ubuntu/Debian
sudo apt-get install ipmitool

# CentOS/RHEL
sudo yum install ipmitool

# macOS
brew install ipmitool
```

### 连接到 BMC 的两种方式

#### 方式 A：本地连接（In-band）
```bash
# 不需要认证，直接访问本地 BMC
ipmitool sensor list          # 查看所有传感器
ipmitool chassis status       # 查看机箱状态
```

#### 方式 B：远程连接（Out-of-band）
```bash
# 需要 BMC 的 IP、用户名、密码
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password <命令>

# 参数说明：
# -I lanplus     ← 使用 IPMI v2.0 over LAN
# -H <IP>        ← BMC 的 IP 地址
# -U <username>  ← 用户名
# -P <password>  ← 密码
```

---

## 4. 常用命令详解

### ① 电源控制
```bash
# 查看电源状态
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power status

# 开机
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power on

# 关机
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power off

# 强制关闭（即刻断电）
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power cycle

# 重启
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power reset

# 软关闭（通知系统优雅关机）
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power soft
```

### ② 硬件监控
```bash
# 查看所有传感器值
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sensor list

# 示例输出：
# Temp1                | 35 degrees C | ok
# Temp2                | 42 degrees C | ok
# CPU Fan              | 5400 RPM     | ok
# System Fan           | 3200 RPM     | ok
# VCORE               | 1.25 V       | ok

# 查看特定传感器
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sensor reading "Temp1"

# 查看传感器详细信息
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sdr list
```

### ③ 查看系统事件日志（SEL）
```bash
# 列出所有硬件事件
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sel list

# 示例输出：
# 1  | 2026-05-18T10:30:00 | Power Supply #1 | Power Supply | Critical
# 2  | 2026-05-18T10:31:00 | Temp1           | Temperature | Warning
# 3  | 2026-05-18T10:32:00 | CPU Fan         | Fan         | Critical

# 清除日志
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sel clear

# 获取 SEL 统计信息
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sel info
```

### ④ 远程控制台（Serial Over LAN）
```bash
# 建立远程 KVM/串口连接
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password -e ~ sol activate

# 参数：
# -e ~     ← 设置退出序列为 ~（按 ~. 退出）

# 断开连接：在 SOL session 中按 ~.
```

### ⑤ 用户管理
```bash
# 列出所有用户
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password user list

# 创建新用户
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password user set name 21 newuser

# 设置用户密码
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password user set password 21 newpassword

# 启用用户
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password user enable 21

# 分配权限（20=管理员、4=操作员、2=用户）
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password channel setaccess 1 21 ipmi=on privilege=4
```

### ⑥ 机箱 LED 控制
```bash
# 查看 LED 状态
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis led status

# 点亮定位 LED（闪烁帮助物理定位服务器）
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis identify 300  # 300秒后自动关闭

# 关闭定位 LED
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis identify 0
```

### ⑦ BMC 网络配置
```bash
# 查看 BMC 网络设置
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password lan print

# 设置 IP 地址为 DHCP
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password lan set 1 ipsrc dhcp

# 设置静态 IP
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password lan set 1 ipsrc static
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password lan set 1 ipaddr 192.168.1.50
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password lan set 1 netmask 255.255.255.0
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password lan set 1 defgw ipaddr 192.168.1.1
```

### ⑧ FRU 信息（库存信息）
```bash
# 查看服务器硬件信息
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password fru print

# 示例输出：
# FRU Device Description : Builtin FRU Device
#  Board Mfg Date/Time : Mon Jan  1 00:00:00 1996
#  Board Product Name : SUPERMICRO
#  Board Serial Number: SM12345678
```

---

## 5. 实际应用场景

### 场景 1：服务器故障排查
```bash
#!/bin/bash
# 检查服务器运行状态

BMC_IP="192.168.1.100"
BMC_USER="admin"
BMC_PASS="password"

IPMI_CMD="ipmitool -I lanplus -H $BMC_IP -U $BMC_USER -P $BMC_PASS"

echo "=== 电源状态 ==="
$IPMI_CMD chassis power status

echo "=== 温度监控 ==="
$IPMI_CMD sensor | grep -i temp

echo "=== 风扇状态 ==="
$IPMI_CMD sensor | grep -i fan

echo "=== 最近事件 ==="
$IPMI_CMD sel list | tail -10
```

### 场景 2：批量远程重启
```bash
#!/bin/bash
# 远程批量重启服务器

for i in {1..10}; do
    IP="192.168.1.$((100+i))"
    echo "重启 $IP ..."
    ipmitool -I lanplus -H $IP -U admin -P password chassis power reset
done
```

### 场景 3：监控告警
```bash
#!/bin/bash
# 持续监控温度，超过阈值发送告警

while true; do
    TEMP=$(ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sensor reading "Temp1" | awk '{print $2}')
    
    if (( $(echo "$TEMP > 70" | bc -l) )); then
        echo "警告：温度过高 $TEMP°C" | mail -s "服务器告警" admin@example.com
    fi
    
    sleep 300  # 每5分钟检查一次
done
```

---

## 6. 常见问题与解决方案

| 问题 | 原因 | 解决方案 |
|------|------|--------|
| 无法连接 BMC | 网络问题或 BMC 关闭 | 检查 BMC IP/网络，重启 BMC |
| 认证失败 | 用户名/密码错误 | 确认凭证，重置 BMC 密码 |
| "Command timed out" | BMC 响应缓慢 | 增加超时时间 `-N 5` |
| 权限不足 | 用户权限不够 | 提升用户权限级别 |
| 找不到传感器 | 硬件不支持或驱动缺失 | 更新 IPMI 驱动或 BMC 固件 |

---

## 7. IPMI 与现代替代品

虽然 IPMI 仍广泛使用，但新服务器逐渐转向：
- **Redfish**：基于 REST API，更易集成自动化
- **iLO/iDRAC/DRAC**：厂商特定的管理接口（基于 IPMI，但功能更多）

许多现代系统同时支持 IPMI 和 Redfish，IPMI 仍是可靠的备选方案。

