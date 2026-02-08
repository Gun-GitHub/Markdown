## 官网主页

```sh
https://github.com/huashengdun/webssh/tree/v1.6.2
```

## 安装

在登录节点的物理机上首先查看有没有 python,版本要大于等于3.6

```sh
[root@mn1 system]# python3 --version
Python 3.6.8
```

然后检查有没有 pip,并且注意这是哪个 python 的 pip

```sh
[root@mn1 system]# pip --version
pip 21.3.1 from /usr/local/lib/python3.6/site-packages/pip (python 3.6)
```

然后安装 webssh

```sh
pip install webssh -i https://pypi.tuna.tsinghua.edu.cn/simple 
```

无外网环境安装,首先通过指令下载安装包:

```sh
pip download webssh -d /path/to/save/package -i https://pypi.tuna.tsinghua.edu.cn/simple 
```

拷贝到物理机上,再本地化安装:

```sh
pip install /path/to/webssh-1.6.2.tar.gz
```

然后创建守护进程脚本:

```sh
cat > /usr/lib/systemd/system/wssh.service << EOF
[Unit]
Description=wssh daemon
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/wssh --xsrf=False --port=9999 --logging=debug --log-file-prefix=/var/log/wssh/wssh.log
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
# Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF
```

创建日志目录:

```sh
mkdir /var/log/wssh
```

服务重载:

```sh
systemctl daemon-reload
```

开机启动:

```sh
systemctl enable wssh.service 
```

开启服务

```sh
systemctl start wssh.service 
```

## 使用

在可视化桌面使用浏览器打开

```sh
http://物理节点的ip地址:9999/?hostname=物理节点的主机名&username=物理节点上的用户&password=base_64加密后的用户密码
```

base_64加密后的用户密码获取方法:

```sh
echo -n "用户密码" | base64
```
