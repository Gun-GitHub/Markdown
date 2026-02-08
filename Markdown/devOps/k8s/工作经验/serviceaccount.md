在 Kubernetes 中，当你挂载一个 ServiceAccount 到 Pod 时，Kubernetes 会在 Pod 的容器内自动注入一些信息，主要是用于 API 访问的凭证。具体来说，挂载的内容通常包括：

1. **ServiceAccount Token**：一个 JWT 令牌，通常位于 `/var/run/secrets/kubernetes.io/serviceaccount/token`。这个令牌用于身份验证。
2. **CA 证书**：用于验证 Kubernetes API 服务器的 SSL 证书，路径为 `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`。
3. **Namespace**：当前 Pod 所在的命名空间，路径为 `/var/run/secrets/kubernetes.io/serviceaccount/namespace`。

这些文件的默认挂载路径是 `/var/run/secrets/kubernetes.io/serviceaccount/`，并且它们会在 Pod 启动时自动创建。你可以在容器中直接访问这些文件来获取认证信息。