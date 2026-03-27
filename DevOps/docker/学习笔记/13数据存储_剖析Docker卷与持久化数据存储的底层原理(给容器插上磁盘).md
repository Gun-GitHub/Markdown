容器按照业务类型,总体可以分为:

1. 无状态的(数据不需要被持久化)
2. 有状态的(数据需要被持久化)

显然容器更擅长无状态应用, 因为被持久化的容器根目录的生命周期与容器的生命周期一样, 容器文件系统的本质是在容器镜像层上面创建读写层,运行中的容器对任何文件的修改都存放于读写层,而容器被删除时, 容器中的读写层也随之消失,虽然容器都希望所有的业务都保持无状态,这样容器就可以开箱即用,并且可以任意调度,但实际业务总是有各种需要数据持久化的场景.比如 mysql, kafka, 等有状态的业务,因此吗为了解决有状态业务的需求, docker 提出了卷的概念

## 卷

卷(Volume) 本质是文件或者目录, 可以绕过默认的联合文件系统, 直接以文件或者目录的形式存在于宿主机上

卷的概念不仅解决了数据持久化的问题,还解决了容器间数据共享的问题 

使用卷可以将容器的目录或者文件持久化, 当容器重启时保证数据不丢失

使用 docker volume 命令可以实现对卷的创建,查看和删除等操作

#### 创建数据卷

```sh
$ docker volume create myvolume
```

默认情况下,  docker 创建的数据卷为 local 模式,仅能提供本主机的容器访问, 如果想要实现远程访问,需要通过网络存储来实现

docker 的 local 模式并未提配额管理, 因此, 在生产环境中, 需要手动维护磁盘存储空间

除了使用 docker volume 的方式创建卷, 还可以在 docker 启动时使用 -v 的方式指定容器内需要被持久化的路径

```sh
$ docker run -d --name=nginx-volume -v /usr/share/nginx/html nginx
```

docker 会自动帮我们创建卷, 并且绑定到容器中

#### 查看主机上的数据卷

```sh
$ docker volume ls
```

查看数据卷的详细信息

```sh
$ docker volume inspect myvolume
[
    {
        "CreatedAt": "2024-11-06T11:49:15+08:00",
        "Driver": "local",
        "Labels": {
            "com.docker.volume.anonymous": ""
        },  
        "Mountpoint": "/data/docker-data/default/volumes/804519e8729545a8fffaa64d1ce6de07382e82ef17ffdb20da8baa38e8e343f0/_data",
        "Name": "myvolume",
        "Options": null,
        "Scope": "local"
    }
]
```

#### 将创建的数据挂载在容器中

```sh
$ docker run -d --name=nginx --mount source=myvolume,target=/usr/share/nginx/html nginx
```

使用 docker 的卷可以将指定目录的数据持久化

容器删除并不会删除已经创建的卷,因此不再使用的数据卷需要我们手动删除

#### 删除数据卷

删除上面个创建的数据卷

```sh
$ docker volume rm myvolume
```

注意, 正在被使用中的数据卷无法被删除, 如果你想要删除使用中的数据卷, 需要删除所有关联的容器

#### 容器与容器之间的数据共享

 实际使用场景:

	容器内产生的日志需要一个专门的日志采集程序去采集日志内容

	例如使用 Filebeat(一种日志采集工具)采集 nginx 容器内的日志

	需要使用卷来共享一个日志目录, 从而使得 Filebeat 和 nginx 容器都可以访问这个目录

首先创建一个共享日志数据卷

```sh
$ docker volume create log-vol
```

启动一个生产日志的容器(下面是用 producer 窗口来表示)

```sh
$ docker run --mount source=log-vol,target=/tmp/log --name=log-producer -it busybox
```

新打开一个命令行窗口,启动一个消费者容器(下面是用 consumer 窗口来表示)

```sh
docker run -it --name consumer --volumes-from log-producer busybox
```

使用 --volumes-from 参数可以在启动新的容器时,来挂载已经存在的容器的卷, --volumes-from 参数后面跟启动容器的名称

,在 log-porducer 容器中创建的容器的日志内容,在 consumer 容器中可以被看见

总结, 使用 docker volume create 命令创建了 log-vol 卷来作为==**共享目录**==, log-producer 容器向该卷写入数据, consumer 容器从该卷读取数据

这就像主机上的两个进程,一个向主机目录写数据,一个从主机目录读数据,利用了主机的目录, 实现了容器之间的数据共享

## Docker 卷目录的位置

Docker 卷的目录默认在 /var/lib/docker 下

## 主机与容器之间的数据共享

当想把主机的其他目录映射到容器内时, 需要用到==主机与容器之间数据共享方式==   
