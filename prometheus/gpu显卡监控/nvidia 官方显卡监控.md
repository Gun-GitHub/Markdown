添加 nvidia DCGM-Exporter helm 仓库:

```sh
helm repo add gpu-helm-charts https://nvidia.github.io/dcgm-exporter/helm-charts
```

更新 helm 仓库:

```sh
helm repo update
```

安装 chart:

```sh
helm install \
    --generate-name \
    gpu-helm-charts/dcgm-exporter
```

### 以上是官方推荐方法,但是 chart 仓库已经 404 了,以下实际部署方法:

1. 下拉官方 dcgm git 仓库:

```sh
git clone https://github.com/NVIDIA/dcgm-exporter.git
```

2. 找到文件夹:

```sh
/deployment
```

3. 修改文件夹的名字:

```sh
mv deployment dcgm-exporter
```

4. 打包:

```sh
helm package dcgm-exporter
```

5. 将打好的包上传到私服 Nexus repo 中的 helm-hpc-dev

6. 添加 helm 库:

```sh
helm add repo helm-hpc-dev http://192.168.1.143:8081/repository/helm-hpc-dev/
```

7. 更新 helm 库:

```sh
helm repo update
```

8. 创建命令空间:

```sh
kubectl create namesapce nvidia-dcgm-exporter
```

9. 部署 dcgm:

```sh
helm install dcgm-exporter helm-hpc-dev/dcgm-exporter -n nvidia-dcgm-exporter
```