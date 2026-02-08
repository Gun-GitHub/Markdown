```sh
cat /etc/containerd/config.toml
...文件内容
                  [plugins."io.containerd.grpc.v1.cri".registry.configs]
    [plugins."io.containerd.grpc.v1.cri".registry.configs."idocker.io".tls] # nexus3地址
        insecure_skip_verify = true
```