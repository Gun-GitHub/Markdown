完成windows端部署后注入promethus的方案

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: windows-exporter-mn1
  namespace: windows-exporter
  labels:
    app: windows-exporter-mn1
subsets:
  - addresses:
      - ip: 192.168.1.201
        nodeName: windows-mn1
    ports:
      - name: windows-exporter-mn1
        port: 9182
        protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: windows-exporter-mn1
  namespace: windows-exporter
  labels:
    app: windows-exporter-mn1
spec:
  ports:
    - name: windows-exporter-mn1
      port: 9182
      protocol: TCP
      targetPort: 9182
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: windows-exporter-mn1
  namespace: windows-exporter
spec:
  selector:
    matchLabels:
      app: windows-exporter-mn1
  namespaceSelector:
    matchNames:
      - windows-exporter
  endpoints:
    - port: windows-exporter-mn1
      scheme: http
      path: /metrics
      scrapeTimeout: 10s
      interval: 20s
      metricRelabelings:
        - replacement: 192.168.1.201
          targetLabel: instance
        - replacement: windows-mn1
          targetLabel: node

```
