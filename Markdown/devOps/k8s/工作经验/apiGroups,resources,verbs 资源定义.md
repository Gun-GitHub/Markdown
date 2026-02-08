在 Kubernetes 中，`ClusterRole` 是一种权限控制资源，它定义了一组访问权限，用于控制对集群范围内资源的访问。`ClusterRole` 中的 `rules` 字段包含了 `apiGroups`、`resources` 和 `verbs` 三个主要部分。以下是这些字段可以填写的内容：

### 1. **`apiGroups`**

   `apiGroups` 定义了 Kubernetes API 组，可以填写以下内容：

- `""`：表示核心 API 组（例如 `pods`、`services` 等）。
- `"apps"`：表示 `apps` API 组（例如 `deployments`、`statefulsets` 等）。
- `"batch"`：表示 `batch` API 组（例如 `jobs`、`cronjobs` 等）。
- `"extensions"`：旧版扩展 API 组，包含一些早期资源类型。
- `"networking.k8s.io"`：表示网络相关的 API 组（例如 `networkpolicies`）。
- `"rbac.authorization.k8s.io"`：表示 RBAC API 组。
- `"apiextensions.k8s.io"`：表示自定义资源定义（CRD）相关的 API 组。
- 以及其他扩展 API 组。

### 2. **`resources`**

   `resources` 定义了具体资源类型，可以填写的内容包括：

- 核心资源：`pods`、`services`、`nodes`、`namespaces`、`configmaps`、`secrets`、`persistentvolumeclaims` 等。
- 应用资源：`deployments`、`statefulsets`、`daemonsets` 等。
- 批处理资源：`jobs`、`cronjobs`。
- 网络资源：`networkpolicies`、`ingresses`。
- 自定义资源：自定义资源定义后产生的资源类型，例如 `crontabs`（假设你定义了一个名为 `crontabs` 的 CRD）。

### 3. **`verbs`**

   `verbs` 定义了可以对资源执行的操作，可以填写以下内容：

- `"get"`：获取资源的详情。
- `"list"`：列出资源。
- `"watch"`：监视资源的变化。
- `"create"`：创建资源。
- `"update"`：更新资源。
- `"patch"`：部分更新资源。
- `"delete"`：删除资源。
- `"deletecollection"`：删除资源集合。
- `"approve"`：批准资源（通常用于 `certificatesigningrequests`）。
- `"bind"`：绑定资源（通常用于 `roles`、`clusterroles`）。
- `"escalate"`：权限升级（通常用于 `roles`、`clusterroles`）。
- `"impersonate"`：模拟用户、组或服务账户。
- `"use"`：使用某些 Kubernetes 资源（例如 `serviceaccounts`、`secrets`）。
- `"get"`、`"list"`、`"create"`、`"update"` 等常见操作通常一起使用。

以下是一个示例配置：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: example-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["create", "update", "patch"]
```

这个配置允许持有该 `ClusterRole` 的用户或服务账户在集群范围内执行与 `pods` 和 `services` 相关的查看操作，以及对 `deployments` 和 `statefulsets` 的创建和更新操作。
