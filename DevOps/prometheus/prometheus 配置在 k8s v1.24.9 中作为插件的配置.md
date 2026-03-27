要在 Kubernetes 中配置 Prometheus 的 ServiceMonitor，使得 Prometheus 中的 **Discovered Labels** 转换成 **Target Labels**，你需要使用 **relabeling** 规则。这些规则允许你在服务发现后对标签进行修改、添加或删除，从而将发现的标签转换为 Prometheus 期望的目标标签。

以下是配置步骤和示例：

### 1. 理解 Relabeling

- **relabelings**: 在服务发现阶段应用，用于修改服务发现时的标签。
- **metricRelabelings**: 在抓取指标阶段应用，用于修改指标的标签。

在你的场景中，通常需要在 `relabelings` 部分进行标签转换。

### 2. 配置 ServiceMonitor

下面是一个示例 `ServiceMonitor` 配置，展示如何将发现的标签转换为目标标签：

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-servicemonitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: your-app
  namespaceSelector:
    matchNames:
      - your-namespace
  endpoints:
  - port: metrics
    relabelings:
    - sourceLabels: [__meta_kubernetes_service_label_custom_label]
      targetLabel: custom_label
    - sourceLabels: [__meta_kubernetes_pod_node_name]
      targetLabel: node
    # 添加更多的 relabeling 规则根据需要转换标签
```

### 3. 详细说明

- **sourceLabels**: 指定从服务发现中获取的原始标签。通常这些标签以 `__meta_kubernetes_` 开头。
- **targetLabel**: 指定要应用于 Prometheus 目标的标签名称。

### 4. 示例解释

假设你的 Kubernetes 服务或 Pod 有一个自定义标签 `custom_label`，你希望将其转换为 Prometheus 的 `custom_label` 标签。在上述示例中：

```yaml
relabelings:
- sourceLabels: [__meta_kubernetes_service_label_custom_label]
  targetLabel: custom_label
```

这条规则将 Kubernetes 服务的 `custom_label` 标签值赋给 Prometheus 目标的 `custom_label` 标签。

### 5. 常见用例

- **重命名标签**：
  ```yaml
  - sourceLabels: [__meta_kubernetes_pod_label_env]
    targetLabel: environment
  ```
  
  将 Pod 的 `env` 标签重命名为 `environment`。
- **过滤目标**：
  
  只抓取具有特定标签值的目标。
  ```yaml
  - sourceLabels: [__meta_kubernetes_service_label_monitor]
    regex: true
    action: keep
  ```
  
  仅保留具有 `monitor=true` 标签的服务。

### 6. 验证配置

完成配置后，确保 Prometheus 能正确发现并抓取目标。你可以通过以下步骤进行验证：

1. **检查 Prometheus Targets**：
访问 Prometheus 的 Web 界面，导航到 **Status > Targets**，查看新配置的目标是否被正确发现并抓取。
2. **查看标签**：
在 Prometheus 的 **Targets** 页面，检查目标的标签是否按照配置进行了转换。

### 7. 参考资料

- [Prometheus Operator 文档 - ServiceMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/api.md#servicemonitor)
- [Prometheus Relabeling 配置](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config)

通过以上配置和步骤，你可以有效地将 Kubernetes 中发现的标签转换为 Prometheus 目标标签，满足监控和告警的需求。

### 8. ServiceMonitor工作机制

ServiceMonitor 的工作机制可以总结为以下步骤：

1. 创建和应用 ServiceMonitor： 用户首先创建一个 ServiceMonitor 对象，并将其应用到 Kubernetes 集群中。ServiceMonitor 的 YAML 配置定义了要监控的服务和相关的监控规则，包括标签选择器、抓取间隔、指标路径等。

2. Prometheus Operator 监控 ServiceMonitor： Prometheus Operator 运行在 Kubernetes 集群中，它会监听 ServiceMonitor 的创建和更新事件。一旦 ServiceMonitor 被创建或更新，Prometheus Operator 就会感知到并对其进行处理。

3. Service 发现与关联： 当 ServiceMonitor 被创建时，Prometheus Operator 会根据 ServiceMonitor 中定义的标签选择器来自动发现符合条件的 Service。ServiceMonitor 和要监控的 Service 是通过标签关联的，通过在 Service 和 ServiceMonitor 的标签选择器中使用相同的标签，将 ServiceMonitor 与目标 Service 相关联起来。

4. Prometheus 指标抓取： 一旦 ServiceMonitor 和 Service 关联起来，Prometheus Operator 会将 ServiceMonitor 中定义的指标抓取规则（包括抓取间隔、指标路径等）配置到 Prometheus 中。Prometheus 使用这些配置信息来定期抓取目标 Service 暴露的指标数据。

5. 指标数据存储和查询： Prometheus 定期抓取 Service 暴露的指标数据，并将其存储在自身的时间序列数据库中。Prometheus 提供一套灵活的查询语言，可以对存储的指标数据进行查询和分析。

6. Grafana 可视化和警报： Prometheus 数据可以被可视化工具如 Grafana 所使用，通过 Grafana 用户可以创建仪表盘来展示监控指标的图表。同时，Prometheus 还支持警报规则的定义，可以在特定条件满足时触发警报通知。
