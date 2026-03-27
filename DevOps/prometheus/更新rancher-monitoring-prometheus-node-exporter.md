# 更新rancher-monitoring-prometheus-node-exporter 

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: rancher-monitoring-prometheus-node-exporter
  namespace: cattle-monitoring-system
spec:
  endpoints:
    - port: http-metrics
      relabelings:
        - sourceLabels:
            - __meta_kubernetes_pod_host_ip
          targetLabel: host_ip
        - sourceLabels:
            - __meta_kubernetes_pod_node_name
          targetLabel: node
      scheme: http
  jobLabel: jobLabel
  selector:
    matchLabels:
      app.kubernetes.io/instance: rancher-monitoring
      app.kubernetes.io/name: prometheus-node-exporter

```