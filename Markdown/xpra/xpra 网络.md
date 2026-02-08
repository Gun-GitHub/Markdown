### **连接类型及其选项**

Xpra 支持多种连接类型，以满足不同场景的需求。以下是一些主要的连接类型及其绑定选项：

1. **TCP (传输控制协议)**: 使用 `bind-tcp` 选项。适用于所有平台，支持通过网络进行连接。
2. **QUIC (快速 UDP 互联网连接)**: 使用 `bind-quic` 选项。适用于所有平台，旨在提供比 TCP 更低的延迟。
3. **SSL (安全套接字层)**: 使用 `bind-ssl` 选项。适用于所有平台，为 TCP 连接提供加密。
4. **SSH (安全外壳协议)**: 使用 `bind-ssh` 选项。适用于所有平台，提供安全的远程登录功能。
5. **WebSocket**: 使用 `bind-ws` 选项。适用于所有平台，允许通过 Web 浏览器进行连接。
6. **Secure WebSocket**: 使用 `bind-wss` 选项。适用于所有平台，为 WebSocket 提供加密。
7. **RFB (远程帧缓冲)**: 使用 `bind-rfb` 选项。仅适用于桌面和阴影服务器，允许 VNC 客户端连接。
8. **Unix 域套接字**: 使用 `bind` 选项。适用于 Posix 系统，用于本地连接或通过 SSH。
9. **命名管道**: 使用 `bind` 选项。仅适用于 MS Windows。
10. **vsock**: 使用 `bind-vsock` 选项。适用于 Linux，用于宿主机与虚拟机间的连接。
    
     TCP 套接字还可以自动升级为 (安全的) WebSocket、SSL、SSH 和 RFB，因此单个 TCP 端口可以自动支持 6 种不同的协议。未加密的模式，如纯 TCP 和纯 WebSocket，也可以使用 AES 加密。
    
     除 vsock 和命名管道外，所有可通过网络连接的套接字通常会通过组播 DNS 发布。在 Posix 上，unix 域套接字作为 SSH 暴露，因为假设本地始终可用 SSH 服务器。

## 例如:

- TCP 升级到 WebSocket 的示例：

```sh
xpra start --start=xterm --bind-tcp=0.0.0.0:10000

```

```sh
xpra attach ws://localhost:10000/
```

```
同样的地址（此处为10000）也可以在浏览器中打开以使用HTML5客户端：
```

```sh
xdg-open [http://localhost:10000/](http://localhost:10000/)
```

- 使用密码文件的 SSH 示例：

```sh
echo -n thepassword > password.txt
xpra start --start=xterm --bind-ssh=0.0.0.0:10000,auth=file:filename=password.txt

```

```sh
xpra attach ssh://localhost:10000/
```

### 网络性能

Xpra 会尝试检测您的网络适配器和连接特性，并应对网络容量和性能的变化进行调整。然而，它可能并不总是做得正确，您可能需要关闭带宽检测（`bandwidth-detection` 选项）和/或指定自己的带宽限制（`bandwidth-limit` 选项）。

您可以使用 Xpra 系统托盘菜单中的“Session Info”对话框的“Graphs”标签页查看带宽使用情况和图片延迟情况：

```sh
`Session Info : Graphs` 
```

更多网络信息可以在“Session Info”对话框的其他部分或通过“xpra info”命令找到,network latency via xpra info :

```sh
`$ xpra info | egrep -i "network|latency"` 
(..)
client.latency.50p=3
client.latency.80p=3
client.latency.90p=3
client.latency.absmin=1
(..)

```

Xpra 的性能将受到您的网络连接速度的影响，特别是缓冲膨胀（bufferbloat）已知会导致严重的性能下降，因为 Xpra 对网络抖动和延迟相当敏感，尝试在您的网络中消除缓冲膨胀。

QUIC 应该提供最低的延迟，尽管它可能需要一些调整。
