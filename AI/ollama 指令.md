# 大语言模型 & RAG

## 资源

### 底层后端: 

ollama(大语言模型部署服务)

```sh
github: https://github.com/ollama/ollama
官网url: https://ollama.com/
```

### 前端: 	

maxkb(基于大语言模型和 RAG 的知识库问答系统)

```sh
github: https://github.com/1Panel-dev/MaxKB
官网url: https://maxkb.cn/
```

## ollama 部署

### 1. docker 部署 ollama

docker 启动 ollama 镜像

```sh
docker run -d --gpus=all -v /dockerdata/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

#### a. 模型文件操作

ollama 拉取模型

```sh
docker exec -it ollama ollama pull llama3.1
```

ollama 删除模型

```
docker exec -it ollama ollama rm qwen3:8b
```

ollama 查询模型

```
docker exec -it ollama ollama list
```

查看所有模型

```sh
docker exec -it ollama ollama list
```

#### b. 模型启停操作

ollama 运行模型

```sh
docker exec -it ollama ollama run llama3.1
```

ollama 查看正在运行中的模型

```sh
docker exec -it ollama ollama ps
```

ollama 停止模型

```sh
docker exec -it ollama ollama stop qwen3:8b
```

### 2. windows 部署 ollama

安装包下载 url:

```sh
https://objects.githubusercontent.com/github-production-release-asset-2e65be/658928958/f69e962c-bf1d-4582-a98a-d5184d64d81d?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction/20241110/us-east-1/s3/aws4_request&X-Amz-Date=20241110T134147Z&X-Amz-Expires=300&X-Amz-Signature=9634a4a05010b33de5cc6fdb26be9d68c0e701c98a332575c794c7d7c37ce382&X-Amz-SignedHeaders=host&response-content-disposition=attachment; filename=OllamaSetup.exe&response-content-type=application/octet-stream
```

双击安装,在隐藏图标中看到羊驼的标识,说明 ollama 已经部署成功了, 或者访问 url:

```sh
http://localhost:11434/
# 看到 ollama is running,就表示 ollama 已经部署成功了
```

windows 的安装默认不支持修改程序安装目录, 默认位置:

```sh
默认安装后的目录：C:\Users\username\AppData\Local\Programs\Ollama
默认安装的模型目录：C:\Users\username\ .ollama
默认的配置文件目录：C:\Users\username\AppData\Local\Ollama
```

由于Ollama的模型默认会在C盘用户文件夹下的.ollama/models文件夹中，可以配置环境变量OLLAMA_MODELS，设置为指定的路径：

![截图](./image_ollama指令/7af0aa6ed35c286dc0b876370a43acbe.png)

ollama 拉取模型

```sh
ollama pull llama3.1
```

ollama 运行模型

```sh
ollama run llama3.1
```

查看所有模型

```sh
ollama ollama list
```

## MaxKB 部署

### MaxKB 不支持直接在 windows 上部署,只能通过 docker 的方式

```
# Linux 操作系统
docker run -d --name=maxkb --restart=always -p 8080:8080 -v ~/.maxkb:/var/lib/postgresql/data -v ~/.python-packages:/opt/maxkb/app/sandbox/python-packages cr2.fit2cloud.com/1panel/maxkb

# Windows 操作系统
docker run -d --name=maxkb --restart=always -p 8080:8080 -v C:/maxkb:/var/lib/postgresql/data -v C:/python-packages:/opt/maxkb/app/sandbox/python-packages cr2.fit2cloud.com/1panel/maxkb

# wsl
docker volume create maxkb
docker volume create maxkb-python-packages
docker run -d --name=maxkb --restart=always -p 8080:8080 -v maxkb:/var/lib/postgresql/data -v maxkb-python-packages:/opt/maxkb/app/sandbox/python-packages cr2.fit2cloud.com/1panel/maxkb
```

### 安装成功后，可通过浏览器访问 MaxKB：

```
http://目标服务器 IP 地址:8080

默认登录信息
用户名：admin
默认密码：MaxKB@123..
```

## 使用 ORAS 将 ollama 模型上传至私服

### 1. ollama 模型存放路径结构

```shell
root@ec78c314b113:~/.ollama/models# tree .
.
|-- blobs
|   |-- sha256-05a61d37b08453e59290add468e3bb2f688e23a01e967fecb0e2fa41218cea76
|   |-- sha256-a3de86cd1c132c822487ededd47a324c50491393e6565cd14bafa40d0b8e686f
|   |-- sha256-ae370d884f108d16e7cc8fd5259ebc5773a0afa6e078b11f4ed7e39a27e0dfc4
|   |-- sha256-cff3f395ef3756ab63e58b0ad1b32bb6f802905cae1472e6a12034e4246fbbdb
|   `-- sha256-d18a5cc71b84bc4af394a31116bd3932b42241de70c77d2b76d69a314ec8aa12
`-- manifests
    `-- registry.ollama.ai
        `-- library
            `-- qwen3
                `-- 8b
root@ec78c314b113:~/.ollama/models# cat manifests/registry.ollama.ai/library/qwen3/8b
{
    "schemaVersion": 2,
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "config": {
        "mediaType": "application/vnd.docker.container.image.v1+json",
        "digest": "sha256:05a61d37b08453e59290add468e3bb2f688e23a01e967fecb0e2fa41218cea76",
        "size": 487
    },
    "layers": [
        {
            "mediaType": "application/vnd.ollama.image.model",
            "digest": "sha256:a3de86cd1c132c822487ededd47a324c50491393e6565cd14bafa40d0b8e686f",
            "size": 5225374496
        },
        {
            "mediaType": "application/vnd.ollama.image.template",
            "digest": "sha256:ae370d884f108d16e7cc8fd5259ebc5773a0afa6e078b11f4ed7e39a27e0dfc4",
            "size": 1723
        },
        {
            "mediaType": "application/vnd.ollama.image.license",
            "digest": "sha256:d18a5cc71b84bc4af394a31116bd3932b42241de70c77d2b76d69a314ec8aa12",
            "size": 11338
        },
        {
            "mediaType": "application/vnd.ollama.image.params",
            "digest": "sha256:cff3f395ef3756ab63e58b0ad1b32bb6f802905cae1472e6a12034e4246fbbdb",
            "size": 120
        }
    ]
}
root@ec78c314b113:~/.ollama/models# cat blobs/sha256-05a61d37b08453e59290add468e3bb2f688e23a01e967fecb0e2fa41218cea76
{
    "model_format": "gguf",
    "model_family": "qwen3",
    "model_families": [
        "qwen3"
    ],
    "model_type": "8.2B",
    "file_type": "Q4_K_M",
    "architecture": "amd64",
    "os": "linux",
    "rootfs": {
        "type": "layers",
        "diff_ids": [
            "sha256:a3de86cd1c132c822487ededd47a324c50491393e6565cd14bafa40d0b8e686f",
            "sha256:eb4402837c7829a690fa845de4d7f3fd842c2adee476d5341da8a46ea9255175",
            "sha256:d18a5cc71b84bc4af394a31116bd3932b42241de70c77d2b76d69a314ec8aa12",
            "sha256:cff3f395ef3756ab63e58b0ad1b32bb6f802905cae1472e6a12034e4246fbbdb"
        ]
    }
}
```

### 2. ORAS 下载

​	[github 地址](https://github.com/oras-project/oras/releases) 下载

```shell
# 在主机中
zhoujun@ubuntu:~/下载$ curl -LO https://github.com/oras-project/oras/releases/download/v1.3.0/oras_1.3.0_linux_amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 4515k  100 4515k    0     0   123k      0  0:00:36  0:00:36 --:--:--  144k


# 在 ollama 容器中
root@ec78c314b113:~# mkdir oras && cd oras
root@ec78c314b113:~/oras# scp zhoujun@192.168.1.22:/home/zhoujun/\344\270\213\350\275\275/oras_1.3.0_linux_amd64.tar.gz .
root@ec78c314b113:~/oras# tar -zxvf oras_1.3.0_linux_amd64.tar.gz
LICENSE
oras
root@ec78c314b113:~/oras# ll
total 15760
drwxr-xr-x 2 root root     4096 Feb 26 06:21 ./
drwx------ 1 root root     4096 Feb 26 04:45 ../
-rw-r--r-- 1 1001  118    11343 Sep  8 00:43 LICENSE
-rwxr-xr-x 1 1001  118 11489428 Sep  3 04:18 oras*
-rw-r--r-- 1 root root  4624108 Feb 26 03:04 oras_1.3.0_linux_amd64.tar.gz
root@ec78c314b113:~/oras# mv oras /usr/local/bin/
root@ec78c314b113:~/oras# oras version
Version:        1.3.0
Go version:     go1.25.0
OS/Arch:        linux/amd64
Git commit:     40530fe4c68e5825b868cd874bd46fc0cdd0f432
Git tree state: clean
```

### 3. 创建 Sonatype Nexus Repository 库

1). 创建 **Blob Stores**

![1](./image_ollama指令/1.png)

2). 创建 **Repositories**

![2](./image_ollama指令/2.png)

### 上传

```sh
oras push 192.168.100.160:8085/ollama-models/qwen3:8b \
  --username admin \
  --password \!\@\#\$1qazasdf \
  --image-spec v1.0 \
  --plain-http \
  --config \
  blobs/sha256-05a61d37b08453e59290add468e3bb2f688e23a01e967fecb0e2fa41218cea76:application/vnd.docker.container.image.v1+json \ # 对应上面的 config.digest 这是 ollama list 识别模型的关键,也是元数据,很重要,不能漏掉
  blobs/sha256-05a61d37b08453e59290add468e3bb2f688e23a01e967fecb0e2fa41218cea76:application/vnd.docker.container.image.v1+json \
  blobs/sha256-a3de86cd1c132c822487ededd47a324c50491393e6565cd14bafa40d0b8e686f:application/vnd.ollama.image.model \
  blobs/sha256-ae370d884f108d16e7cc8fd5259ebc5773a0afa6e078b11f4ed7e39a27e0dfc4:application/vnd.ollama.image.template \
  blobs/sha256-d18a5cc71b84bc4af394a31116bd3932b42241de70c77d2b76d69a314ec8aa12:application/vnd.ollama.image.license \
  blobs/sha256-cff3f395ef3756ab63e58b0ad1b32bb6f802905cae1472e6a12034e4246fbbdb:application/vnd.ollama.image.params \
  manifests/registry.ollama.ai/library/qwen3/8b:application/vnd.ollama.image.manifests # 对应上面的 manifests/registry.ollama.ai/library/qwen3/8b 很重要不能漏掉
```

### 下拉

```
oras pull --plain-http 192.168.100.160:8085/ollama-models/qwen3:8b
```

## 
