#手动安装参考
https://github.com/docker/buildx
二进制拷贝并命名至/root/.docker/cli-plugins/docker-buildx

### kernel 3.10 异构docker build测试

```
docker buildx ls
docker buildx create mybuilder --platform arm .. <略>
docker buildx use mybuilder

docker buildx build --platform linux/arm64 -f Dockerfile-frontend-arm64 -t idocker.io/hpc_frontend:1.0-arm . --load

#测试构建结果是否正常
#注册qemu，由于内核版本过低(3.10) 无法使用-p yes直接写到宿主机的某目录中/proc/sys/fs/binfmt_misc
#只能用稍微麻烦一点的办法
docker run --rm --privileged multiarch/qemu-user-static:register --reset

$ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
enabled

interpreter /usr/bin/qemu-aarch64-static
flags: 
offset 0
magic 7f454c460201010000000000000000000200b700
mask ffffffffffffff00fffffffffffffffffeffffff
$ docker create -it --name dummy multiarch/qemu-user-static:x86_64-aarch64 bash
$ docker container ls -a
CONTAINER ID        IMAGE                                       COMMAND             CREATED             STATUS              PORTS               NAMES
6ab622a76dfa        multiarch/qemu-user-static:x86_64-aarch64   "bash"              3 minutes ago       Created                                 dummy
$ docker cp dummy:/usr/bin/qemu-aarch64-static qemu-aarch64-static
$ ls qemu-aarch64-static
qemu-aarch64-static*
$ docker rm -f dummy
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
#成功输出架构信息
[root@mn1 root_script]# docker run --rm -t -v $(pwd)/qemu-aarch64-static:/usr/bin/qemu-aarch64-static idocker.io/hpc_jdk_juice_base:1.0-arm uname -m
aarch64

```