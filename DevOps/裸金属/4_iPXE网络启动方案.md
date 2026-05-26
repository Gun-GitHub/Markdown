## 📍 知识导航
- 🔗 [[0_裸金属知识体系|← 返回知识体系]]
- 📚 相关文档: [[BMC远程拉取操作系统]] | [[5_ipxe指令说明]]
- 👓 本文是对 [[BMC远程拉取操作系统#七、iPXE + HTTP 无盘安装|BMC 中 iPXE 方案]] 的详细展开

---

```mermaid
%%{init: {
  "flowchart": {
    "nodeSpacing": 3,
    "rankSpacing": 30,
    "defaultRenderer": "elk"
  }
}}%%
%%{init: {
  "themeVariables": {
    "nodePadding": 1
  }
}}%%
flowchart TD
	
	subgraph Server[操作系统分发服务器]
		Service1[DHCP服务]
		Service2[TFTP服务]
		Service3[HTTP服务]
		Service4[Redfish管理服务]
		style Service1 fill:#f00
		style Service2 fill:#f00
		style Service3 fill:#f00
		style Service4 fill:#f00
		
		S_content1[[指定TFTP服务器地址]]
		S_content2[(提供 IPXE.efi/snponly.efi,并提供启动脚本 autoexec.ipxe/boot.ipxe)]
		S_content3[(提供操作系统 ubuntu.iso, 并提供操作系统内核)]
		S_content4[[通过redfish接口对服务器进行操作]]
		style S_content1 fill:#bfb,stroke:#333,stroke-width:2px
		style S_content2 fill:#bfb,stroke:#333,stroke-width:2px
		style S_content3 fill:#bfb,stroke:#333,stroke-width:2px
		style S_content4 fill:#bfb,stroke:#333,stroke-width:2px
		
		prot1((端口68))
		prot2((端口69))
		prot3((端口8888/任意))
		prot4((端口任意))
		style prot1 fill:#ff0,stroke:#333,stroke-width:2px
		style prot2 fill:#ff0,stroke:#333,stroke-width:2px
		style prot3 fill:#ff0,stroke:#333,stroke-width:2px
		style prot4 fill:#ff0,stroke:#333,stroke-width:2px
		
		S_content1 --> Service1 --> prot1 --> NC1[计算网卡ens18]
		S_content2 --> Service2 --> prot2 --> NC1[计算网卡ens18]
		S_content3 --> Service3 --> prot3 --> NC1[计算网卡ens18]
		
		S_content4 --> Service4 --> prot4 --> NC2[管理网卡ens18]
	end
	
	subgraph N1[计算网络]
		NC1 ---> N1_Router
		N1_Router[路由器]
		
		N1_Switch1[交换机]
		N1_Switch2[交换机]
		
		N1_content1[关闭DHCP服务]
		N1_content2[关闭DHCP服务]
		N1_content3[关闭DHCP服务]
		style N1_content1 fill:#bfb,stroke:#333,stroke-width:2px
		style N1_content2 fill:#bfb,stroke:#333,stroke-width:2px
		style N1_content3 fill:#bfb,stroke:#333,stroke-width:2px
		
		N1_content1 --> N1_Router
		
		N1_content2 --> N1_Switch1
		N1_Router --> N1_Switch1
		
		N1_Router --> N1_Switch2
		N1_content3 --> N1_Switch2
	end
	
	subgraph N2[管理网络]
		NC2 --> N2_Router
		N2_Router[路由器]
		
		N2_Switch1[交换机]
		N2_Switch2[交换机]
	
		N2_Router --> N2_Switch1
		N2_Router --> N2_Switch2
	end
	
	subgraph C[计算节点层]
		subgraph CN1[CN1]
			BMC1[[BMC/IPMI]]
			PXE1[[PXE系统引导]]
			
			CN-BMC-NC1[BMC网卡]
			CN-NC1[计算网卡]
			BIOS1[主板]
			
			OS1[[操作系统]]
			
			CN-BMC-NC1 --> BMC1 --> BIOS1 --> PXE1
			CN-NC1 --> PXE1 --> OS1
		end
		subgraph CN2[CN2]
			BMC2[[BMC/IPMI]]
			PXE2[[PXE系统引导]]
			
			CN-BMC-NC2[BMC网卡]
			CN-NC2[计算网卡]
			BIOS2[主板]
			
			OS2[[操作系统]]
			
			CN-BMC-NC2 --> BMC2 --> BIOS2 --> PXE2
			CN-NC2 --> PXE2 --> OS2
		end
		subgraph CN3[CN3]
			BMC3[[BMC/IPMI]]
			PXE3[[PXE系统引导]]
			
			CN-BMC-NC3[BMC网卡]
			CN-NC3[计算网卡]
			BIOS3[主板]
			
			OS3[[操作系统]]
			
			CN-BMC-NC3 --> BMC3 --> BIOS3 --> PXE3
			CN-NC3 --> PXE3 --> OS3
		end
		subgraph CN4[CN4]
			BMC4[[BMC/IPMI]]
			PXE4[[PXE系统引导]]
			
			CN-BMC-NC4[BMC网卡]
			CN-NC4[计算网卡]
			BIOS4[主板]
			
			OS4[[操作系统]]
			
			CN-BMC-NC4 --> BMC4 --> BIOS4 --> PXE4 
			CN-NC4 --> PXE4 --> OS4
		end
	
		N2_Switch1 --> CN-BMC-NC1
		N2_Switch1 --> CN-BMC-NC2
		N2_Switch2 --> CN-BMC-NC3
		N2_Switch2 --> CN-BMC-NC4
		
		N1_Switch1 ----> CN-NC1
		N1_Switch1 ----> CN-NC2
		N1_Switch2 ----> CN-NC3
		N1_Switch2 ----> CN-NC4
	end
	
	linkStyle 12 stroke:#10B981,stroke-width:4px
	linkStyle 15 stroke:#10B981,stroke-width:4px
	linkStyle 16 stroke:#10B981,stroke-width:4px
	
	linkStyle 18 stroke:#3B82F6,stroke-width:4px
	linkStyle 19 stroke:#3B82F6,stroke-width:4px
	linkStyle 20 stroke:#3B82F6,stroke-width:4px
	
	
	linkStyle 45 stroke:#10B981,stroke-width:4px
	linkStyle 46 stroke:#10B981,stroke-width:4px
	linkStyle 47 stroke:#10B981,stroke-width:4px
	linkStyle 48 stroke:#10B981,stroke-width:4px
	
	linkStyle 41 stroke:#3B82F6,stroke-width:4px
	linkStyle 42 stroke:#3B82F6,stroke-width:4px
	linkStyle 43 stroke:#3B82F6,stroke-width:4px
	linkStyle 44 stroke:#3B82F6,stroke-width:4px
	
	style Server fill:#BFDBFE,stroke:#1E3A8A,stroke-width:2px
	style CN1 fill:#BFDBFE,stroke:#1E3A8A,stroke-width:2px
	style CN2 fill:#BFDBFE,stroke:#1E3A8A,stroke-width:2px
	style CN3 fill:#BFDBFE,stroke:#1E3A8A,stroke-width:2px
	style CN4 fill:#BFDBFE,stroke:#1E3A8A,stroke-width:2px
```

# 一. 搭建 IPXE 服务器
## i. 部署 DHCP 服务
1. 安装 isc-dhcp-server tftpd-hpa
```bash
apt install isc-dhcp-server
```

2. 配置 /etc/dhcp/dhcpd.conf 如下
```bash
(base) root@mn1:~# cat /etc/dhcp/dhcpd.conf 
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.100 192.168.100.200;
    option routers 192.168.100.1;
    option broadcast-address 192.168.1.255;
 
    next-server 192.168.100.160; # <=============== 这里要指向 TFTP 所在服务器的 IP
    filename "ipxe.efi";   # UEFI机器
}
```

3. 检查 DHCP 服务是否配置正确
```bash
(base) root@mn1:~# dhcpd -t -cf /etc/dhcp/dhcpd.conf
Internet Systems Consortium DHCP Server 4.4.3-P1
Copyright 2004-2022 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
```

4. 启动 DHCP 服务
```bash
systemctl restart isc-dhcp-server
```

## ii. 部署 TFTP 服务
1. 安装 TFTP服务
```bash
apt install tftpd-hpa
```

2. 配置 /etc/default/tftpd-hpa 如下:
```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
#TFTP_DIRECTORY="改成你存放 ipxe.efi 的路径"
TFTP_DIRECTORY="/var/ty-root/users/zhoujun/Downloads"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"
```

3. 启动 tftp 服务
```bash
systemctl restart tftpd-hpa.service 
```

4. 下载 ipxe.efi
```bash
# 项目源码地址
https://github.com/ipxe/ipxe
# 项目编译后的地址
https://github.com/ipxe/ipxe/releases
# 下载指令
wget https://github.com/ipxe/ipxe/releases/download/v2.0.0/ipxeboot.tar.gz
# 解压 tar 包
tar -zxvf ipxeboot.tar.gz
# 解压项目结构
.
├── arm32
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   └── snponly.efi
├── arm64
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   └── snponly.efi
├── arm64-sb
│   ├── ipxe.efi
│   ├── ipxe-shim.efi -> shimaa64.efi
│   ├── shimaa64.efi
│   ├── snponly.efi
│   └── snponly-shim.efi -> shimaa64.efi
├── i386
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   ├── ipxe-legacy.pxe
│   ├── ipxe.pxe
│   ├── snponly.efi  # <=================================================企业服务器推荐使用这个
│   └── undionly.kpxe
├── ipxe.efi -> x86_64/ipxe.efi
├── ipxe-legacy.efi -> x86_64/ipxe-legacy.efi
├── ipxe-legacy.pxe -> x86_64/ipxe-legacy.pxe
├── ipxe.pxe -> x86_64/ipxe.pxe
├── loong64
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   └── snponly.efi
├── riscv32
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   └── snponly.efi
├── riscv64
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   └── snponly.efi
├── sb -> x86_64-sb
├── snponly.efi -> x86_64/snponly.efi
├── undionly.kpxe -> x86_64/undionly.kpxe
├── x86_64
│   ├── ipxe.efi
│   ├── ipxe-legacy.efi
│   ├── ipxe-legacy.pxe
│   ├── ipxe.pxe
│   ├── snponly.efi
│   └── undionly.kpxe
└── x86_64-sb
    ├── ipxe.efi
    ├── ipxe-shim.efi -> shimx64.efi
    ├── shimx64.efi
    ├── snponly.efi
    └── snponly-shim.efi -> shimx64.efi

11 directories, 43 files
```

5. 编辑操作系统引导脚本 autoexec.ipxe
```ipxe
#!ipxe
dhcp
chain http://192.168.100.160:8888/boot.ipxe
```
与 snponly.efi 存放在同一目录下

6. 编辑系统启动脚本 boot.ipxe
```ipxe
#!ipxe
dhcp
set base http://192.168.100.160:8888
kernel ${base}/vmlinuz ip=dhcp boot=casper netboot=http url=${base}/ubuntu-26.04-live-server-amd64.iso
initrd ${base}/initrd
boot
```
与 snponly.efi 存放在同一目录下

## iii. 部署 HTTP 服务
1. 下载操作系统 iso
```bash
wget https://releases.ubuntu.com/26.04/ubuntu-26.04-live-server-amd64.iso
```

2. 获取操作系统内核
```bash
mkdir /mnt/iso
mount -o loop ubuntu-26.04-live-server-amd64.iso /mnt/iso/
cp /mnt/iso/casper/vmlinuz ./vmlinuz
cp /mnt/iso/casper/initrd ./initrd
```

3. 将所有在的这些都放到 http 服务器提供获取的路径下

4. 启动 http 服务
```bash
python3 -m http.server 8888 # 端口号要和 boot.ipxe 中的 base 写的一样
```

## iv. 部署 Redfish 管理服务
1. 设置PXE启动
```bash
curl --location 'https://192.168.100.34/redfish/v1/Systems/1' \
--header 'Content-Type: application/json' \
-k -u Administrator:Admin@9000 \
--data '{
    "Boot": {
        "BootSourceOverrideEnabled": "Once",
        "BootSourceOverrideTarget": "Pxe"
    }
}'
```

2. 开机/关机
```bash
curl --location 'https://192.168.100.34/redfish/v1/Systems/1/Actions/ComputerSystem.Reset' \
--header 'Content-Type: application/json' \
-k -u Administrator:Admin@9000 \
# 开机
--data '{
  "ResetType": "On"
}'
# 正常关机
--data '{
  "ResetType": "GracefulShutdown"
}'
# 强制关机
--data '{
  "ResetType": "ForceOff"
}'
# 强制重启
--data '{
  "ResetType": "ForceRestart"
}'
# 正常重启
--data '{
  "ResetType": "GracefulRestart"
}'
```

