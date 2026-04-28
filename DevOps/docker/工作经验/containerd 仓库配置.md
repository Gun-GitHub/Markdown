[[docker]]
```sh
cat /etc/containerd/config.toml
...文件内容
                  [plugins."io.containerd.grpc.v1.cri".registry.configs]
    [plugins."io.containerd.grpc.v1.cri".registry.configs."idocker.io".tls] # nexus3地址
        insecure_skip_verify = true
```

---

### 关联笔记
- [[学习笔记/11组件组成-剖析 Docker 组件作用及其底层工作原理|11组件组成-剖析 Docker 组件作用及其底层工作原理]]
- [[学习笔记/5仓库访问-怎样搭建属于你的私有仓库|5仓库访问-怎样搭建属于你的私有仓库]]
- [[部署常用的工具/部署SonatypeNexusRepository|部署SonatypeNexusRepository]]
