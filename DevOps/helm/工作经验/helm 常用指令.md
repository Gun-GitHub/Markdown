### 创建新 chart 的命令:

```sh
helm create chart_name
```

### 测试新构建的 chart 指令:

```sh
cd chart_dir && helm install resource_name --dry-run --debug --set key=value .
```

### 添加 helm 库:

```sh
helm add repo $HELM_REPO $HELM_REPO_ADDR
```

### 更新 helm 仓库:

```sh
helm repo update
```

### 检查 helm 仓库信息:

```shell
helm repo list
```

### 使用指定的 config 文件

```shell
# 使用 --kubeconfig 参数（推荐、最明确）
helm --kubeconfig /path/to/kubeconfig list
# 使用环境变量 KUBECONFIG
export KUBECONFIG=/path/to/kubeconfig
helm list
```

法的萨菲
