# Redfish 详细指南

## 目录
- [1. Redfish 概述](#1-redfish-概述)
- [2. Redfish 架构](#2-redfish-架构)
- [3. 核心特性](#3-核心特性)
- [4. 环境准备](#4-环境准备)
- [5. 基础使用](#5-基础使用)
- [6. 常见操作](#6-常见操作)
- [7. 进阶应用](#7-进阶应用)
- [8. 安全最佳实践](#8-安全最佳实践)
- [9. 故障排查](#9-故障排查)
- [10. 参考资源](#10-参考资源)

---

## 1. Redfish 概述

### 1.1 什么是 Redfish？

**Redfish** 是由 DMTF（分布式管理任务组 - Distributed Management Task Force）开发的现代**服务器硬件管理标准**，提供了一套基于 **REST API** 的规范，用于管理和监控数据中心的物理和虚拟基础设施。

### 1.2 Redfish vs IPMI

| 特性 | IPMI | Redfish |
|------|------|---------|
| 协议 | RMCP/RMCP+ | REST/HTTPS |
| 接口 | 命令行 | RESTful API |
| 加密 | 支持，但繁琐 | 原生支持 HTTPS |
| 易用性 | 学习曲线陡峭 | 现代、易于集成 |
| 跨平台 | 平台差异大 | 标准化、统一 |
| 扩展性 | 有限 | 高度可扩展 |
| 认证 | 基本认证 | OAuth 2.0, Kerberos |
| 状态码 | 仅数字代码 | HTTP 状态码 |
| 异步操作 | 不支持 | 完全支持 |

### 1.3 Redfish 的优势

- ✅ **现代标准**：基于 REST/JSON，与现代开发工具兼容
- ✅ **安全性**：强制 HTTPS，支持现代认证机制
- ✅ **可编程性**：易于集成到自动化工具链中
- ✅ **统一接口**：不同厂商服务器有一致的接口
- ✅ **可扩展**：支持厂商特定的扩展（OEM）
- ✅ **标准化**：符合国际标准规范

---

## 2. Redfish 架构

### 2.1 架构层级

```
┌─────────────────────────────────────────┐
│     应用层 (Management Applications)     │
│  编排系统、监控工具、自动化框架等        │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│     API 层 (Redfish REST API)            │
│     /redfish/v1/...                      │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│     传输层 (HTTPS/TLS 1.2+)             │
│     加密安全通信                          │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│     BMC 层 (Baseboard Management Ctrl)   │
│     服务器底板管理控制器                  │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│     硬件层 (Physical Hardware)           │
│     CPU, Memory, Power, 网络等          │
└─────────────────────────────────────────┘
```

### 2.2 关键组件

| 组件 | 说明 |
|------|------|
| **BMC** (Baseboard Mgmt Controller) | 独立的微处理器，管理服务器硬件 |
| **iLO/iDRAC/XClarity** | 不同厂商的 BMC 实现 |
| **Redfish Service** | BMC 上运行的 API 服务 |
| **Event Service** | 事件和告警推送服务 |
| **Task Service** | 异步任务执行和追踪 |
| **Schema** | API 响应的数据结构定义 |

---

## 3. 核心特性

### 3.1 服务器发现

- 自动发现和识别服务器资源
- 兼容所有主流厂商（HP、Dell、IBM/Lenovo、华为等）

### 3.2 系统管理

- **电源管理**：开/关/重启/安全关闭
- **固件升级**：BIOS、BMC、驱动更新
- **系统配置**：BIOS 设置、启动顺序、网络配置

### 3.3 监控和告警

- **实时监控**：CPU、内存、温度、电源状态
- **传感器数据**：温度、风速、电压、功耗等
- **事件告警**：推送关键告警到管理系统

### 3.4 用户和安全管理

- **用户管理**：创建、删除、权限分配
- **访问控制**：基于角色的权限（RBAC）
- **审计日志**：记录所有操作日志

### 3.5 虚拟媒体

- 挂载远程 ISO 镜像进行系统安装或故障排查

### 3.6 远程控制台

- **文本控制台**：Serial-over-LAN (SOL)
- **图形控制台**：远程视频、键盘、鼠标

---

## 4. 环境准备

### 4.1 硬件要求

- **支持的服务器**：HP ProLiant、Dell PowerEdge、IBM System x 等
- **BMC 固件**：需要支持 Redfish 的 BMC 版本

### 4.2 网络要求

```
管理工作站
    ↓
    └─→ 管理网络（VLAN 隔离）
            ↓
        ┌────────────────┐
        │   服务器 BMC    │
        │  (HTTPS:443)   │
        └────────────────┘
```

- BMC 需要独立的网络接口（1 Gbps 推荐）
- 支持 IPv4 和 IPv6

### 4.3 软件工具

#### Python 方案
```bash
# 安装 Redfish Python 库
pip install redfish

# 或安装增强工具
pip install redfish-utilities
```

#### Curl/HTTP 工具
```bash
# 系统自带
curl --version
wget --version

# 或安装 HTTPie
pip install httpie
```

#### 开源工具
- **Sushy**：OpenStack Redfish 客户端
- **ipmitool**：IPMI/Redfish 兼容工具
- **ipmitool-fru**：查看 FRU 信息

### 4.4 配置 BMC

#### 4.4.1 网络配置

```bash
# 1. 连接 BMC 管理端口（通常单独的网口）
# 2. 登录 Web 界面进行初始配置

# 示例（HP iLO）
https://ilo-ip
用户名：Administrator
密码：Administrator
```

#### 4.4.2 HTTPS 证书

- 使用默认自签名证书（测试）
- 配置正式 SSL 证书（生产）

---

## 5. 基础使用

### 5.1 认证和连接

#### 方式一：使用 curl

```bash
# 基本认证
curl -i -u Administrator:password \
  --insecure \
  https://192.168.1.100/redfish/v1/

# 响应示例
HTTP/1.1 200 OK
Content-Type: application/json
{
  "@odata.type": "#ServiceRoot.v1_16_0.ServiceRoot",
  "@odata.id": "/redfish/v1/",
  "Id": "RootService",
  "Name": "Root Service"
}
```

#### 方式二：使用 Python

```python
import redfish

# 连接到 BMC
conn = redfish.connect_using_bmc_credentials(
    "https://192.168.1.100",
    "Administrator",
    "password"
)

# 或使用标准连接
conn = redfish.redfish_client(
    base_url="https://192.168.1.100",
    username="Administrator",
    password="password",
    default_prefix="/redfish/v1"
)

# 验证连接
print(conn.get_session_location())

# 记得断开连接
conn.logout()
```

#### 方式三：使用 HTTPie

```bash
# 安装
pip install httpie

# 请求
http --auth Administrator:password \
     --verify=no \
     https://192.168.1.100/redfish/v1/
```

### 5.2 API 路径结构

```
/redfish/v1/                          # 根服务
├── /Systems                          # 系统集合
│   └── /{SystemId}                   # 特定系统
│       ├── /Processors               # CPU 集合
│       ├── /Memory                   # 内存集合
│       ├── /Storage                  # 存储集合
│       ├── /EthernetInterfaces       # 网络接口
│       ├── /Bios                     # BIOS 设置
│       └── /Actions                  # 可执行操作
│
├── /Chassis                          # 机箱集合
│   └── /{ChassisId}                  # 特定机箱
│       ├── /Sensors                  # 传感器集合
│       ├── /Power                    # 电源状态
│       ├── /Thermal                  # 温度信息
│       └── /Actions                  # 可执行操作
│
├── /Managers                         # BMC 集合
│   └── /{ManagerId}                  # 特定 BMC
│
└── /EventService                     # 事件服务
```

### 5.3 常见查询

#### 查看系统信息

```bash
# 查看所有系统
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Systems

# 查看特定系统详情
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Systems/1
```

#### 查看硬件配置

```python
import redfish

conn = redfish.redfish_client(
    base_url="https://192.168.1.100",
    username="Administrator",
    password="password"
)

# 获取系统列表
systems = conn.get("/redfish/v1/Systems")
for system in systems.get("Members", []):
    system_url = system["@odata.id"]
    system_data = conn.get(system_url)
    
    print(f"系统: {system_data['Id']}")
    print(f"制造商: {system_data.get('Manufacturer')}")
    print(f"型号: {system_data.get('Model')}")
    print(f"序列号: {system_data.get('SerialNumber')}")
    print(f"电源状态: {system_data.get('PowerState')}")
    print(f"启动方式: {system_data.get('BootSourceOverrideMode')}")

conn.logout()
```

---

## 6. 常见操作

### 6.1 电源管理

#### 6.1.1 电源操作

```bash
# 定义操作 URL
POWER_ACTION="https://192.168.1.100/redfish/v1/Systems/1/Actions/ComputerSystem.Reset"

# 启动服务器
curl -X POST -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{"ResetType": "On"}' \
  $POWER_ACTION

# 关闭服务器
curl -X POST -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{"ResetType": "Off"}' \
  $POWER_ACTION

# 重启服务器
curl -X POST -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{"ResetType": "GracefulRestart"}' \
  $POWER_ACTION

# 强制重启
curl -X POST -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{"ResetType": "ForceRestart"}' \
  $POWER_ACTION
```

#### 6.1.2 查看电源状态

```bash
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Chassis/1/Power | jq
```

### 6.2 BIOS 配置

#### 6.2.1 查看 BIOS 设置

```bash
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Systems/1/Bios | jq
```

#### 6.2.2 修改 BIOS 设置

```bash
# 修改单个设置
curl -X PATCH -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{
    "Attributes": {
      "BootMode": "Uefi",
      "NumaMode": "Enabled"
    }
  }' \
  https://192.168.1.100/redfish/v1/Systems/1/Bios/Settings
```

### 6.3 固件升级

```python
import redfish

conn = redfish.redfish_client(
    base_url="https://192.168.1.100",
    username="Administrator",
    password="password"
)

# 上传固件文件
with open("firmware.bin", "rb") as f:
    firmware_data = f.read()

# 创建更新任务
update_service = conn.get("/redfish/v1/UpdateService")
print(f"固件版本: {update_service.get('FirmwareVersion')}")

# 提交固件更新请求
response = conn.post(
    "/redfish/v1/UpdateService/Actions/UpdateService.SimpleUpdate",
    body={
        "ImageURI": "http://192.168.1.50/firmware.bin",
        "TransferProtocol": "HTTP"
    }
)

print(f"更新任务: {response.get('Task')}")
conn.logout()
```

### 6.4 用户管理

#### 6.4.1 查看用户列表

```bash
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/AccountService/Accounts | jq
```

#### 6.4.2 创建新用户

```bash
curl -X POST -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{
    "UserName": "newuser",
    "Password": "SecurePassword123!",
    "RoleId": "Operator",
    "Enabled": true
  }' \
  https://192.168.1.100/redfish/v1/AccountService/Accounts
```

#### 6.4.3 修改用户密码

```bash
curl -X PATCH -u Administrator:password \
  --insecure \
  -H "Content-Type: application/json" \
  -d '{
    "Password": "NewPassword123!"
  }' \
  https://192.168.1.100/redfish/v1/AccountService/Accounts/2
```

### 6.5 监控和传感器

#### 6.5.1 温度监控

```python
import redfish

conn = redfish.redfish_client(
    base_url="https://192.168.1.100",
    username="Administrator",
    password="password"
)

# 获取热能信息
thermal = conn.get("/redfish/v1/Chassis/1/Thermal")

print("温度传感器信息:")
for sensor in thermal.get("Temperatures", []):
    print(f"  {sensor.get('Name')}: "
          f"{sensor.get('ReadingCelsius')}°C "
          f"(阈值: {sensor.get('UpperThresholdCritical')}°C)")

print("\n风速传感器信息:")
for fan in thermal.get("Fans", []):
    print(f"  {fan.get('Name')}: "
          f"{fan.get('Reading')} RPM "
          f"(状态: {fan.get('Status', {}).get('State')})")

conn.logout()
```

#### 6.5.2 功耗监控

```bash
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Chassis/1/Power | jq '.PowerSupplies'
```

### 6.6 事件和日志

#### 6.6.1 查看系统日志

```bash
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Systems/1/LogServices/Sel | jq
```

#### 6.6.2 订阅事件（webhook）

```python
import redfish

conn = redfish.redfish_client(
    base_url="https://192.168.1.100",
    username="Administrator",
    password="password"
)

# 创建事件订阅
response = conn.post(
    "/redfish/v1/EventService/Subscriptions",
    body={
        "Name": "MySubscription",
        "Destination": "http://192.168.1.50:8080/webhook",
        "EventTypes": ["Alert", "ResourceAdded", "ResourceRemoved"],
        "Protocol": "Redfish"
    }
)

print(f"订阅创建成功: {response.get('Id')}")
conn.logout()
```

---

## 7. 进阶应用

### 7.1 批量操作脚本

```python
#!/usr/bin/env python3
"""Redfish 批量管理脚本"""

import redfish
import json
from typing import List, Dict

class RedifishManager:
    def __init__(self, host: str, username: str, password: str):
        self.conn = redfish.redfish_client(
            base_url=f"https://{host}",
            username=username,
            password=password
        )
    
    def get_system_info(self) -> Dict:
        """获取系统信息"""
        systems = self.conn.get("/redfish/v1/Systems")
        system_data = self.conn.get(systems["Members"][0]["@odata.id"])
        
        return {
            "id": system_data["Id"],
            "model": system_data.get("Model"),
            "manufacturer": system_data.get("Manufacturer"),
            "power_state": system_data.get("PowerState"),
            "serial_number": system_data.get("SerialNumber")
        }
    
    def get_temperatures(self) -> List[Dict]:
        """获取温度信息"""
        thermal = self.conn.get("/redfish/v1/Chassis/1/Thermal")
        temps = []
        
        for sensor in thermal.get("Temperatures", []):
            temps.append({
                "name": sensor.get("Name"),
                "current": sensor.get("ReadingCelsius"),
                "warning": sensor.get("UpperThresholdWarning"),
                "critical": sensor.get("UpperThresholdCritical")
            })
        
        return temps
    
    def power_control(self, action: str) -> bool:
        """电源控制"""
        valid_actions = ["On", "Off", "GracefulRestart", "ForceRestart"]
        
        if action not in valid_actions:
            raise ValueError(f"无效的操作: {action}")
        
        try:
            self.conn.post(
                "/redfish/v1/Systems/1/Actions/ComputerSystem.Reset",
                body={"ResetType": action}
            )
            return True
        except Exception as e:
            print(f"操作失败: {e}")
            return False
    
    def logout(self):
        """断开连接"""
        self.conn.logout()


# 使用示例
if __name__ == "__main__":
    manager = RedifishManager(
        host="192.168.1.100",
        username="Administrator",
        password="password"
    )
    
    # 获取系统信息
    info = manager.get_system_info()
    print(json.dumps(info, indent=2, ensure_ascii=False))
    
    # 获取温度
    temps = manager.get_temperatures()
    for temp in temps:
        print(f"{temp['name']}: {temp['current']}°C")
    
    manager.logout()
```

### 7.2 监控系统集成

#### 与 Prometheus 集成

```python
"""Redfish Prometheus Exporter"""
from prometheus_client import start_http_server, Gauge, Counter
import redfish
import time

# 定义指标
temperature_gauge = Gauge('server_temperature_celsius',
                         'Server temperature',
                         ['sensor_name'])
power_gauge = Gauge('server_power_watts',
                   'Power consumption',
                   ['psu_name'])
status_gauge = Gauge('server_power_state',
                    'Power state (1=On, 0=Off)',
                    ['system'])

class RedifishExporter:
    def __init__(self, host, user, passwd):
        self.conn = redfish.redfish_client(
            base_url=f"https://{host}",
            username=user,
            password=passwd
        )
    
    def collect_metrics(self):
        """收集指标"""
        # 温度
        thermal = self.conn.get("/redfish/v1/Chassis/1/Thermal")
        for temp in thermal.get("Temperatures", []):
            temperature_gauge.labels(
                sensor_name=temp.get("Name")
            ).set(temp.get("ReadingCelsius", 0))
        
        # 功耗
        power = self.conn.get("/redfish/v1/Chassis/1/Power")
        for psu in power.get("PowerSupplies", []):
            power_gauge.labels(
                psu_name=psu.get("Name")
            ).set(psu.get("LastPowerOutputWatts", 0))
        
        # 电源状态
        systems = self.conn.get("/redfish/v1/Systems")
        system = self.conn.get(systems["Members"][0]["@odata.id"])
        power_state = 1 if system.get("PowerState") == "On" else 0
        status_gauge.labels(system="system_1").set(power_state)

if __name__ == "__main__":
    exporter = RedifishExporter(
        "192.168.1.100",
        "Administrator",
        "password"
    )
    
    start_http_server(8000)
    
    while True:
        exporter.collect_metrics()
        time.sleep(30)
```

### 7.3 与 Ansible 集成

```yaml
---
# roles/redfish-management/tasks/main.yml
- name: 获取服务器信息
  community.general.redfish_info:
    baseuri: "{{ redfish_host }}"
    username: "{{ redfish_user }}"
    password: "{{ redfish_pass }}"
    category: Systems
  register: system_info

- name: 显示系统信息
  debug:
    msg: "{{ system_info.redfish_facts.system_details }}"

- name: 电源操作 - 开机
  community.general.redfish_command:
    baseuri: "{{ redfish_host }}"
    username: "{{ redfish_user }}"
    password: "{{ redfish_pass }}"
    command: PowerOn
    resource_id: "1"

- name: 等待系统启动
  wait_for:
    host: "{{ inventory_hostname }}"
    port: 22
    state: started
    timeout: 300

- name: 修改 BIOS 设置
  community.general.redfish_config:
    baseuri: "{{ redfish_host }}"
    username: "{{ redfish_user }}"
    password: "{{ redfish_pass }}"
    bios_attributes:
      - { name: "BootMode", value: "Uefi" }
      - { name: "NumaMode", value: "Enabled" }
```

---

## 8. 安全最佳实践

### 8.1 认证和访问控制

✅ **要做**
```bash
# 1. 使用强密码
# - 最少 12 字符
# - 包含大小写字母、数字、特殊字符
# - 定期轮换（每 90 天）

# 2. 启用 HTTPS
# - 使用有效的 SSL 证书
# - 禁用 HTTP

# 3. 启用双因素认证（如支持）
# - 与活动目录集成
# - 使用 OAuth 2.0

# 4. 限制访问
# - 配置 IP 白名单
# - 限制并发会话数

# 5. 定期审计
# - 查看审计日志
# - 删除未使用的账户
```

❌ **不要做**
```bash
# 1. 不使用默认密码
# 2. 不在 HTTP 上传输凭证
# 3. 不在脚本中硬编码密码
# 4. 不从互联网直接暴露 BMC
# 5. 不允许未授权的 API 调用
```

### 8.2 网络隔离

```
┌──────────────────────────┐
│   管理员工作站            │
│   (VPN/堡垒机)           │
└────────────┬─────────────┘
             │
        ┌────▼────┐
        │ VPN/VPC │
        └────┬────┘
             │
    ┌────────▼────────┐
    │ 管理网络 VLAN    │
    │ (10.100.0.0/16) │
    └────────┬────────┘
             │
    ┌────────▼────────┐
    │  服务器 BMC      │
    │ (隔离网络)       │
    └─────────────────┘
```

### 8.3 加密通信

```bash
# 检查 TLS 版本
openssl s_client -connect 192.168.1.100:443 -tls1_2

# 验证证书
curl --cacert ca.crt \
  https://192.168.1.100/redfish/v1/

# 生成自签名证书（测试用）
openssl req -x509 -newkey rsa:4096 \
  -keyout key.pem -out cert.pem -days 365
```

### 8.4 审计和日志

```python
import redfish
import logging

# 配置日志
logging.basicConfig(
    filename='/var/log/redfish-audit.log',
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

class AuditedRedfish:
    def __init__(self, host, user, passwd):
        self.conn = redfish.redfish_client(
            base_url=f"https://{host}",
            username=user,
            password=passwd
        )
        logger.info(f"用户 {user} 连接到 {host}")
    
    def power_control(self, action):
        """记录所有电源操作"""
        logger.warning(f"执行电源操作: {action}")
        
        try:
            self.conn.post(
                "/redfish/v1/Systems/1/Actions/ComputerSystem.Reset",
                body={"ResetType": action}
            )
            logger.info(f"电源操作成功: {action}")
        except Exception as e:
            logger.error(f"电源操作失败: {action} - {e}")
            raise
```

---

## 9. 故障排查

### 9.1 常见错误

| 错误 | 原因 | 解决 |
|------|------|------|
| `Connection refused` | BMC 服务未运行 | 重启 BMC 或检查网络连接 |
| `Unauthorized (401)` | 认证失败 | 检查用户名/密码 |
| `SSL certificate error` | 证书问题 | 使用 `--insecure` 或安装证书 |
| `Timeout` | 网络延迟 | 检查网络连接，增加超时时间 |
| `Not Supported` | 不支持的操作 | 检查 BMC 版本和权限 |

### 9.2 诊断工具

```bash
# 测试连接
curl -v --insecure https://192.168.1.100/

# 测试认证
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/

# 追踪 HTTP 请求
curl -v --trace /tmp/trace.txt \
  -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/Systems

# 验证 JSON 格式
curl -s -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/ | jq '.'

# 查看原始 HTTP 头
curl -i -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/
```

### 9.3 日志收集

```bash
#!/bin/bash
# 收集 Redfish 诊断信息

echo "=== 系统信息 ==="
date
uname -a

echo "=== 网络连接测试 ==="
ping -c 4 192.168.1.100
netstat -an | grep 443

echo "=== BMC 可达性 ==="
curl -I --insecure https://192.168.1.100

echo "=== 认证测试 ==="
curl -u Administrator:password --insecure \
  https://192.168.1.100/redfish/v1/ 2>&1 | head -20

echo "=== 系统日志 ==="
dmesg | tail -20

echo "=== 防火墙规则 ==="
firewall-cmd --list-all 2>/dev/null || iptables -L -n 2>/dev/null
```

### 9.4 性能优化

```python
# 并发请求优化
import redfish
import concurrent.futures

def query_system(system_id):
    conn = redfish.redfish_client(
        base_url="https://192.168.1.100",
        username="Administrator",
        password="password"
    )
    
    system = conn.get(f"/redfish/v1/Systems/{system_id}")
    conn.logout()
    return system

# 批量查询
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(query_system, i) for i in range(1, 10)]
    results = [f.result() for f in concurrent.futures.as_completed(futures)]

print(f"查询了 {len(results)} 个系统")
```

---

## 10. 参考资源

### 10.1 官方规范

- 📖 [DMTF Redfish 规范](https://www.dmtf.org/standards/redfish)
- 📖 [Redfish Schema](https://redfish.dmtf.org/schemas/v1/)
- 📖 [OpenAPI 文档](https://openapi.redfish.dmtf.org/)

### 10.2 厂商实现

| 厂商 | BMC | 支持版本 | 文档 |
|------|------|---------|------|
| HP | iLO | v5.0+ | https://www.hpe.com/us/en/servers/ilo.html |
| Dell | iDRAC | v9.0+ | https://www.dell.com/en-us/dt/idrac/ |
| IBM/Lenovo | XClarity | v3.0+ | https://www.lenovo.com/us/en/ |
| 华为 | iBMC | v2.3+ | https://www.huawei.com/ |
| 浪潮 | M5 | v1.0+ | https://www.inspur.com/ |

### 10.3 开源项目

```bash
# Python 库
pip install redfish                  # 官方库
pip install sushy                    # OpenStack 项目
pip install python-redfish           # 另一个实现

# 工具
git clone https://github.com/DMTF/python-redfish-utility

# 集成
# - Ansible: community.general.redfish_*
# - Terraform: providers/redfish
# - Prometheus: redfish-exporter
```

### 10.4 学习资源

- 🎓 DMTF 在线培训
- 🎓 厂商认证课程
- 📺 YouTube 技术讲座
- 📚 GitHub 示例代码

### 10.5 常用命令速查

```bash
# 基本信息
curl -u user:pass --insecure https://host/redfish/v1/

# 系统
curl -u user:pass --insecure https://host/redfish/v1/Systems
curl -u user:pass --insecure https://host/redfish/v1/Systems/1

# 机箱
curl -u user:pass --insecure https://host/redfish/v1/Chassis

# 管理器
curl -u user:pass --insecure https://host/redfish/v1/Managers

# 电源
curl -u user:pass --insecure https://host/redfish/v1/Chassis/1/Power

# 温度
curl -u user:pass --insecure https://host/redfish/v1/Chassis/1/Thermal

# 用户
curl -u user:pass --insecure https://host/redfish/v1/AccountService/Accounts

# 事件
curl -u user:pass --insecure https://host/redfish/v1/EventService
```

---

## 总结

| 项目 | 说明 |
|------|------|
| **何时使用** | 需要现代化、标准化的服务器管理解决方案 |
| **主要优势** | REST API、HTTPS 安全、跨厂商统一、易于自动化 |
| **典型场景** | 数据中心自动化、云平台集成、监控告警、固件升级 |
| **学习成本** | 低（REST/JSON 基础） |
| **部署难度** | 低（仅需网络连接） |

**快速开始三步**：
1. 配置 BMC 网络和用户账户
2. 使用 curl 或 Python 测试连接
3. 集成到现有的自动化工具链

---

*最后更新: 2026年5月18日*
