```sh
(base) [root@mn1 mydateset]# openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -nooutCertificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 8208740940399713150 (0x71eb4e64bdbb1b7e)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Oct 13 02:24:39 2023 GMT
            Not After : Oct 12 02:24:39 2024 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b5:5d:61:e5:e4:23:78:bc:3a:4e:48:c9:0b:07:
                    60:dd:27:95:51:de:5a:31:bb:5f:ab:33:43:26:b3:
                    27:76:28:fc:f6:1d:ed:c0:e3:d8:d1:cd:35:98:21:
                    50:78:72:92:bf:8d:a0:59:b5:c4:cc:38:f0:59:d6:
                    fd:c2:61:43:e9:03:59:45:00:00:6f:8b:7b:e7:69:
                    ad:b8:5e:3e:a2:89:87:7e:08:3b:3b:3e:3b:35:b1:
                    1e:9e:e6:a0:40:43:c2:66:9e:21:fe:f5:f7:6d:c9:
                    bb:a3:31:cd:5e:c0:da:26:97:b8:c5:6f:17:34:86:
                    b2:d5:f3:41:cd:bc:18:8e:43:37:36:42:2c:18:4f:
                    78:83:a2:0e:6e:76:72:eb:8d:8e:7f:18:41:c2:b2:
                    a9:ed:21:43:eb:1e:ee:8d:38:18:84:1e:e7:21:d3:
                    1e:3a:96:ec:69:da:3b:6d:f8:5b:82:fb:74:cf:11:
                    70:25:86:7d:e3:8a:be:4c:a7:cb:48:63:8a:4d:f3:
                    9e:b8:f9:8e:2c:59:f6:27:dd:48:b9:47:bd:46:42:
                    66:f4:6f:2c:25:cf:c3:e9:65:af:45:86:a6:59:2a:
                    05:9d:f3:3a:1e:81:6e:21:f1:03:31:22:8b:15:52:
                    bb:9e:ef:e1:f7:b1:8b:4c:bd:e0:93:01:22:80:7f:
                    27:df
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Subject Alternative Name: 
                DNS:mn1, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:lb.kubesphere.local, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:localhost, DNS:lb.kubesphere.local, DNS:mn1, DNS:mn1.cluster.local, DNS:cn1, DNS:cn1.cluster.local, DNS:cn2, DNS:cn2.cluster.local, IP Address:10.233.0.1, IP Address:192.168.1.143, IP Address:127.0.0.1, IP Address:192.168.1.143, IP Address:192.168.1.142, IP Address:192.168.1.141, IP Address:10.233.0.1
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        a2:50:0a:35:6d:f9:40:72:bd:a3:3b:b6:b5:00:a6:73:b5:55:
        05:f9:cd:71:bd:d4:20:2c:93:92:5b:ad:11:d4:2c:dd:d9:bc:
        a2:21:e7:af:b5:16:8b:a9:11:25:d2:05:29:4f:c1:cf:fc:b4:
        51:4a:99:16:d1:fa:12:71:cc:b3:6d:4d:87:4e:67:17:6e:21:
        9f:b8:af:61:49:4b:48:b2:8a:5e:fa:ba:6c:c3:94:df:7c:c9:
        d7:29:a1:dd:05:5c:d6:f3:54:5a:f4:81:79:b4:1f:16:ca:ce:
        7f:92:59:af:85:6b:bc:18:10:0e:e9:08:64:76:3d:5a:2b:19:
        17:16:33:8a:dc:72:cc:f1:65:70:95:ab:50:61:fd:e4:27:be:
        a8:66:3d:4f:5b:26:f3:8a:76:38:e6:8a:4b:91:0b:59:6f:5b:
        17:a8:dc:7a:e0:93:ba:bc:a6:56:62:8a:70:34:b9:ed:a5:5e:
        58:fb:24:9c:a0:f1:dc:88:9d:f4:bc:8f:d9:18:41:7a:23:5a:
        68:a6:85:67:a0:21:51:e7:82:e6:ec:5d:b5:6f:e5:32:b1:a8:
        25:0d:1a:36:7a:44:b0:65:fb:27:32:b5:f0:45:3e:63:b4:20:
        a2:8e:24:df:35:8a:3d:a9:41:85:88:0d:40:15:ca:dc:27:ac:
        a1:c5:59:db
```

这是一段 X.509 证书的内容，常用于加密通信中的身份验证（例如 Kubernetes 中的 API 服务器）。我们逐一解析每个字段的含义：

### 1. **Version: 3 (0x2)**

- 证书版本号。此处显示为版本 3（`0x2`），即 X.509 v3 证书。

### 2. **Serial Number: 8208740940399713150 (0x71eb4e64bdbb1b7e)**

- 证书的唯一序列号，用于在同一 CA 签发的证书中区分个体。每个证书都有唯一的序列号。

### 3. **Signature Algorithm: sha256WithRSAEncryption**

- 使用的签名算法。此证书使用 `sha256WithRSAEncryption`，表示通过 RSA 加密算法对 SHA-256 哈希的签名。

### 4. **Issuer: CN = kubernetes**

- 颁发证书的实体（CA）。`CN = kubernetes` 表示该证书是由 Kubernetes 自身的 CA 签发的。

### 5. **Validity**

- **Not Before**: `Oct 13 02:24:39 2023 GMT`
  - 证书的生效时间。此证书从 `2023 年 10 月 13 日 02:24:39 (GMT)` 开始生效。
- **Not After**: `Oct 12 02:24:39 2024 GMT`
  - 证书的到期时间。此证书在 `2024 年 10 月 12 日 02:24:39 (GMT)` 后失效。

### 6. **Subject: CN = kube-apiserver**

- 证书的拥有者信息。`CN = kube-apiserver` 表示证书属于 Kubernetes 的 API 服务器。

### 7. **Subject Public Key Info**

- 公钥信息，包括公钥算法和公钥。
- **Public Key Algorithm**: `rsaEncryption`
  - 公钥加密算法为 RSA。
- **Public-Key: (2048 bit)**
  - RSA 公钥的长度为 2048 位。
- **Modulus**: RSA 公钥的模数，这是公钥的一部分。
- **Exponent**: 公钥的指数，通常为 `65537 (0x10001)`，这是 RSA 公钥的标准指数。

### 8. **X509v3 extensions**

- X.509 版本 3 的扩展字段。
- **X509v3 Key Usage: critical**
  - 表明该字段是 `critical`，即必须满足这些要求。此扩展字段指定证书允许的用途：
    - **Digital Signature**: 数字签名，表明证书可用于签署数据。
    - **Key Encipherment**: 密钥加密，表明证书可以用来加密对称密钥。
- **X509v3 Extended Key Usage**: 
  - 扩展的密钥使用，指定该证书可用于特定用途。
    - **TLS Web Server Authentication**: 表明该证书可用于 TLS（传输层安全）协议中的 Web 服务器身份验证。
- **X509v3 Subject Alternative Name**:
  - 主题备用名称，列出证书可以使用的其他主机名或 IP 地址。此处列出了 Kubernetes 集群中的多种 DNS 名称和 IP 地址，包括：
    - `DNS:mn1`，`DNS:kubernetes.default`，`IP Address:192.168.1.143` 等。
  - 这些名称允许集群中的多个组件使用相同的证书进行通信。

### 9. **Signature Algorithm: sha256WithRSAEncryption**

- 签名算法，与之前相同，使用 RSA 加密对 SHA-256 哈希值签名。

### 10. **Signature Value**

- 签名值：使用证书颁发机构（CA）的私钥对该证书的数据进行的数字签名。这是为了确保证书的完整性，防止篡改。

### 证书结构总结：

- 该证书由 Kubernetes 自己签发，用于 API 服务器，支持 TLS Web 服务器身份验证。
- 证书有效期为 2023 年 10 月 13 日到 2024 年 10 月 12 日。
- 公钥使用 RSA 加密，密钥长度为 2048 位。
- 主题备用名称包括 Kubernetes 集群中的各种主机名和 IP 地址，支持集群内部通信。

# 老版本 k8s 证书更新

在 Kubernetes 1.17 版本中，如果集群证书即将过期或已经过期，可以通过 `kubeadm` 工具重新生成证书。Kubernetes 的证书有效期通常为一年左右，过期后会影响集群的正常运行。

以下是重新生成证书的步骤：

### 1. 备份现有证书

在执行任何操作之前，建议备份现有证书和配置文件：

```sh
sudo cp -r /etc/kubernetes/pki /etc/kubernetes/pki-backup
sudo cp /etc/kubernetes/admin.conf /etc/kubernetes/admin.conf-backup
```

### 2. 使用 `kubeadm certs renew` 命令更新证书

在 Kubernetes 1.17 中，`kubeadm certs renew` 命令还未引入。你可以手动删除过期证书，然后重新生成新证书。通过执行以下步骤手动更新证书：

首先删除旧的控制平面证书：

```sh
sudo rm -f /etc/kubernetes/pki/apiserver.crt /etc/kubernetes/pki/apiserver.key
sudo rm -f /etc/kubernetes/pki/apiserver-kubelet-client.crt /etc/kubernetes/pki/apiserver-kubelet-client.key
sudo rm -f /etc/kubernetes/pki/front-proxy-client.crt /etc/kubernetes/pki/front-proxy-client.key
sudo rm -f /etc/kubernetes/pki/etcd/peer.crt /etc/kubernetes/pki/etcd/peer.key
sudo rm -f /etc/kubernetes/pki/etcd/healthcheck-client.crt /etc/kubernetes/pki/etcd/healthcheck-client.key
```

然后使用 `kubeadm init phase` 重新生成证书：

```sh
sudo kubeadm init phase certs all --config /etc/kubernetes/kubeadm-config.yaml
```

其中 `/etc/kubernetes/kubeadm-config.yaml` 是集群初始化时的配置文件，如果没有该文件，可以跳过 `--config` 选项，直接使用默认配置生成。

### 3. 更新 `kubeconfig` 文件

证书更新后，你需要重新生成 `admin.conf`、`kubelet.conf`、`controller-manager.conf` 和 `scheduler.conf` 文件：

```sh
sudo kubeadm init phase kubeconfig all
```

### 4. 重启组件

重新生成证书后，Kubernetes 的各个组件需要重新加载证书。可以通过重启 `kubelet` 来使组件使用新证书：

```sh
sudo systemctl restart kubelet
```

### 5. 验证证书更新

确认证书已经更新并生效：

```sh
kubeadm certs check-expiration
```

如果使用的是 1.17 版本，可以手动检查证书过期时间：

```sh
openssl x509 -noout -dates -in /etc/kubernetes/pki/apiserver.crt
```

这样可以确保证书已经被正确更新。
