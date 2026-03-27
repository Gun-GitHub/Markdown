1. 下载 containerd 1.6.x tar文件

```shell
wget https://github.com/containerd/containerd/releases/download/v1.6.26/containerd-1.6.26-linux-amd64.tar.gz
# 解压
tar -xzvf containerd-1.6.26-linux-amd64.tar.gz

# 安装二进制文件到 /usr/local/bin
sudo cp bin/* /usr/local/bin/

# 验证版本
/usr/local/bin/containerd --version
```

2. 修改/var/lib/kubelet/kubeadm-flags.env文件

```shell
vim /var/lib/kubelet/kubeadm-flags.env

#"... --container-runtime-endpoint=unix:///var/run/cri-dockerd.sock ..."
#修改为 --container-runtime-endpoint=unix:///run/containerd/containerd.sock
```

3. 创建container 配置文件

```shell
#创建 containerd 配置目录
mkdir -p /etc/containerd
vim /etc/containerd/config.toml
```

config.toml配置如下

```toml
version = 2
root = "/var/lib/containerd"
state = "/run/containerd"

[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0

[debug]
  level = "info"

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
    max_container_log_line_size = 16384
    
    [plugins."io.containerd.grpc.v1.cri".containerd]
      snapshotter = "overlayfs"
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
            SystemdCgroup = true
    
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = [
                "https://idocker.io",
                "https://docker.xuanyuan.me",
                "https://docker.m.daocloud.io",
                "https://dockerproxy.com",
                "https://docker.mirrors.ustc.edu.cn",
                "https://docker.nju.edu.cn",
                "https://registry-1.docker.io"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."idocker.io"]
          endpoint = ["http://idocker.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."idocker.io".tls]
          insecure_skip_verify = true  # 跳过证书验证
```

4. 修改 /etc/systemd/system/containerd.service

```toml
# ExecStart=/usr/bin/containerd 修改
ExecStart=/usr/local/bin/containerd
```

 

5. 修改 /etc/crictl.yaml

```toml
# 修改 unix:///var/run/cri-dockerd.sock
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
```

6. 创建符号链接

```shell
containerd --version
# 输出
# containerd github.com/containerd/containerd v1.6.26 3dd1e886e55dd695541fdcd67420c2888645a495

# 如不是，需要创建符号链接
systemctl stop containerd

# 或者直接替换二进制文件（先备份）
mv /usr/bin/containerd /usr/bin/containerd.bak
ln -s /usr/local/bin/containerd /usr/bin/containerd

# 同样处理其他相关二进制
mv /usr/bin/containerd-shim /usr/bin/containerd-shim.bak
ln -s /usr/local/bin/containerd-shim /usr/bin/containerd-shim

mv /usr/bin/containerd-shim-runc-v2 /usr/bin/containerd-shim-runc-v2.bak
ln -s /usr/local/bin/containerd-shim-runc-v2 /usr/bin/containerd-shim-runc-v2

# 验证
ls -la /usr/bin/containerd*
```

7. 重启k8s相关服务

```shell
systemctl stop kubelet
systemctl stop containerd

systemctl daemon-reload
systemctl start containerd
systemctl start kubelet

# 重建 NVIDIA Device Plugin
kubectl delete pod -n kube-system -l name=nvidia-device-plugin-ds

# 检查 GPU 资源
kubectl describe node gpu2 | grep -A 5 "nvidia.com/gpu"
```

