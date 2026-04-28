[[协议]]
# WebRTC

## 概述

WebRTC（Web Real-Time Communication）是 **W3C API + IETF 协议** 的集合体，使浏览器/应用之间无需插件即可实现点对点（P2P）实时音视频通信和数据传输。

**协议栈层次：**
```
┌────────────────────────────────────────────────────┐
│     WebRTC API (JavaScript)                        │
│  getUserMedia | RTCPeerConnection | RTCDataChannel │
├────────────────────────────────────────────────────┤
│     SDP 协商 (Session Description)                  │
├────────────────────────────────────────────────────┤
│  ICE | STUN | TURN (NAT 穿透/连接建立)               │
├────────────────────────────────────────────────────┤
│  DTLS (密钥交换/加密)                                │
├────────────────────────────────────────────────────┤
│  SRTP (媒体加密传输) | SCTP (数据通道)                │
├────────────────────────────────────────────────────┤
│  UDP (传输层)                                       │
└────────────────────────────────────────────────────┘
```

---

### 核心 API

| API | 作用 |
|-----|------|
| `getUserMedia()` | 获取摄像头/麦克风等媒体流 |
| `RTCPeerConnection` | 管理与对等端的连接、编码协商、媒体传输 |
| `RTCDataChannel` | 在 P2P 通道上传输任意数据（非媒体） |

---

### 信令（Signaling）

WebRTC 本身**不规定**信令协议，由应用层自定义实现（常用 WebSocket / SIP / XMPP）。信令负责交换两类元数据：

1. **SDP（Session Description Protocol）** — 描述双方的媒体能力（编解码器、分辨率、传输地址等）
2. **ICE Candidate** — 网络连接候选地址（IP + 端口）

**典型流程（Offer/Answer 模型）：**
```
Peer A                     Signaling Server                     Peer B
   │                              │                               │
   │── createOffer() ──────────►  │                               │
   │── setLocalDescription(offer) │                               │
   │                              │── 转发 offer ───────────────►  │
   │                              │                               │── setRemoteDescription(offer)
   │                              │                               │── createAnswer()
   │                              │◄── 转发 answer ─────────────   │── setLocalDescription(answer)
   │◄── setRemoteDescription(ans) │                               │
   │                              │                               │
   │──── 交换 ICE candidates ────► │◄──── 交换 ICE candidates ──── │
   │                              │                               │
```

**SDP 示例片段：**
```
v=0
o=- 827784982034516459 2 IN IP4 127.0.0.1
s=-
t=0 0
a=group:BUNDLE 0
a=ice-ufrag:UFJFQ+RL8H2+WFDAkSWm+AAB
a=ice-pwd:aZZEwPXx3f0QYRVl7BOZRZ9r
a=fingerprint:sha-256 8A:D7:B1:AE:E2:54:...
m=video 51372 RTP/AVP 96
a=rtpmap:96 H264/90000
```

---

### ICE 连接建立

ICE（Interactive Connectivity Establishment）用于在各种网络环境下找到两端之间的最佳通信路径。

#### 候选者类型（ICE Candidate Types）

| 类型 | 优先级 | 来源 |
|------|--------|------|
| **Host**（主机候选） | 126 | 直接使用本地网卡 IP |
| **Peer Reflexive**（对端反射） | 110 | 连接检查过程中动态发现 |
| **Server Reflexive**（反射候选） | 100 | 通过 STUN 服务器获取公网 IP |
| **Relay**（中继候选） | 0 | 通过 TURN 服务器中转 |

**优先级计算公式（RFC 8445）：**
```
priority = (2^24) * type_preference +
           (2^8)  * local_preference +
           (256 - component_id)
```

#### STUN / TURN

| 服务器 | 作用 | 适用场景 |
|--------|------|----------|
| **STUN** | 帮助客户端发现自己的公网 IP 和端口 | NAT 穿透 |
| **TURN** | 中继转发媒体数据 | 对称 NAT / 防火墙无法穿透时回退 |

> 公共 STUN 示例：`stun:stun.l.google.com:19302`

**ICE 连通性检查流程：**
1. 双方收集各自的候选者列表
2. 通过信令交换候选者
3. 各自对候选者按优先级排序
4. 按序互相发送 STUN Binding Request 进行连通性检查
5. 选择第一个验证通过的候选者对作为连接路径
6. 如果所有直连尝试失败，回退到 TURN 中继

---

### 安全与加密

WebRTC 强制加密，所有数据必须经过以下两级保护：

1. **DTLS（Datagram TLS）** — 在 UDP 上实现的 TLS，用于交换加密密钥
2. **SRTP（Secure RTP）** — 使用 DTLS 协商的密钥对 RTP 媒体流进行 AES 加密

> "All WebRTC components **must** be encrypted." — RFC 8827

---

### WebRTC ↔ RTP ↔ H.264 的关系

| 层次 | 说明 |
|------|------|
| **WebRTC** | 建立 P2P 连接，协商媒体参数，管理会话生命周期 |
| **RTP** | 承载实际的音频/视频数据包，提供时序、丢检测、SSRC 标识 |
| **H.264** | 视频编码格式，NALU 结构嵌套在 RTP Payload 中传输（支持 FU-A 分片） |

当 WebRTC 使用 H.264 编码时，视频数据经过以下链路：
```
摄像头 → H.264 编码器 → NALU 打包 → RTP 分片(FU-A) → SRTP 加密 → UDP → P2P
```

`PT = 96`（动态负载类型）通常在 SDP 中映射到 H.264，实际传输时 RTP Header 中的 Payload Type 字段即为协商后的值。

---

<br/>

# RTP(Real-time Transport Protocol) 

```
  0               1               2               3             4
  0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |V=2|P|X| CC  |M|     PT        |       sequence number         |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                           timestamp                           |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |           synchronization source (SSRC) identifier            |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |            contributing source (CSRC) identifiers             |
 |                             ....                              |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                      RTP payload data                         |
 |                             ....                              |

```

📌字段详解

|字段|大小|说明|
|--|--|--|
|`V`|2 bits|RTP 版本，固定为 `2`|
|`P`|1 bit|Padding 标志位（是否有额外填充字节）|
|`X`|1 bit|Extension 标志位，是否有扩展头部|
|`CC`|4 bits|CSRC 的数量（最多 15 个）|
|`M`|1 bit|Marker 标志位，常用于帧边界标记|
|`PT`|7 bits|Payload Type，标识负载类型（如 H.264 是 96 动态负载）|
|`sequence number`|16 bits|包序号，用于丢包检测与重排序|
|`timestamp`|32 bits|时间戳，单位取决于负载类型（如视频常用 90kHz）|
|`SSRC`|32 bits|同步源标识符，用于区分多个数据源|
|`CSRC`|0\~15 \* 32 bits|贡献源 ID（可选）|
|`Payload`|不定长|实际媒体数据（如一段 H.264 的 NALU）|

# h264

H.264 NALU 基本结构（无 RTP 时）

H.264 视频由若干个 NALU（Network Abstraction Layer Unit） 构成，每个 NALU 前面有一个起始码。

```
h264 包结构
 0               1               2               3             4
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Start Code Prefix                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      00        |      00       |      00       |      01      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                NALU = NAL Header + NAL Payload                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                              ......                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Start Code Prefix                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      00        |      00       |      00       |      01      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                NALU = NAL Header + NAL Payload                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


其中 NALU 结构
 0               1               2               3             4
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   NAL Header  |                  NAL Payload                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|F|NRI|   Type  |                    ......                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

F (1 bit): forbidden_zero_bit（始终为 0）

NRI (2 bits): NAL reference IDC（重要性标识）

Type (5 bits): NAL 单元类型，如：

|Type|名称|描述|
|--|--|--|
|0|Reserved|保留，未使用|
|1|**Non-IDR slice**|普通帧的一部分（P/B 帧）|
|2|Slice data (partition A)|不常用，slice 数据分区 A（FMO）|
|3|Slice data (partition B)|不常用，slice 数据分区 B|
|4|Slice data (partition C)|不常用，slice 数据分区 C|
|5|**IDR slice (关键帧)**|**关键帧**（I 帧），独立解码，不依赖其他帧|
|6|SEI|补充增强信息（如 HDR、时序信息）|
|7|**SPS (Sequence Parameter Set)**|序列参数集，描述分辨率、帧率等全局解码信息（需先于帧传送）|
|8|**PPS (Picture Parameter Set)**|图像参数集，通常与 SPS 配套|
|9|AUD (Access Unit Delimiter)|表示视频帧（访问单元）的开始|
|10|End of Sequence|不常用，视频序列结束|
|11|End of Stream|不常用，视频流结束|
|12|Filler Data|填充数据，用于码率控制|
|13–18|Reserved|保留，将来使用|
|19|Coded slice of auxiliary picture|辅助图像（如 alpha 通道）|
|20–23|Reserved|保留|

# FU-A 分片（Fragmentation Unit A）

当 NALU 太大时 NALU（> MTU），使用 FU-A 格式分片传输。

```
 0               1               2               3             4
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  FU Indicator |   FU header   |        FU-A playload          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|F|NRI| type=28 |S|E|R|   type  |        FU-A playload          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

S: Start (1) — 表示这是分片开始

E: End (1) — 表示这是分片结束

R: Reserved (0)

Type: 原始 NALU 的 type 值（如 1, 5）
