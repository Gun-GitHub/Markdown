官方页面：

```
https://github.com/prometheus-community/windows_exporter/releases
```

下载 windows exporter:

```
https://release-assets.githubusercontent.com/github-production-release-asset/66566232/3cc24458-790c-488c-b908-f8359239dfac?sp=r&sv=2018-11-09&sr=b&spr=https&se=2025-07-21T02:35:31Z&rscd=attachment;+filename=windows_exporter-0.31.1-amd64.msi&rsct=application/octet-stream&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2025-07-21T01:34:32Z&ske=2025-07-21T02:35:31Z&sks=b&skv=2018-11-09&sig=hz6m8QIGVc8xAS7OjP7lOrbYuGIkmOqgvNdY2MJEZUk=&jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc1MzA2MjM2NywibmJmIjoxNzUzMDYyMDY3LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.y0-_KaoPlqiXoqGWNNQT6M9EAWlZliWhKX1st0K-h1E&response-content-disposition=attachment; filename=windows_exporter-0.31.1-amd64.msi&response-content-type=application/octet-stream
```

注册表路径：
![截图](8d71354cc3a11b15929c4a4c30f94545.png)

无法设置安装路径，默认安装路径为：

```
C:\Program Files\windows_exporter
```

默认配置文件在：

```
C:\Program Files\windows_exporter\config.yaml
```

在对应的 windows 节点上双击安装：

设置：--web.listen-address="0.0.0.0:9182"

![截图](d2672dabb8ff43ffc2b6cabc9d9753ac.png)

然后一路next

在当前机器浏览器测试：

```
http://<当前机器ip>:9182/metrics
http://192.168.1.202:9182/metrics
```

能访问到如下数据：

![截图](817e61f99dc8ee80665a52f8dc6e4c7c.png)

即为部署成功
