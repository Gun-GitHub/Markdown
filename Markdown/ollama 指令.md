# 大语言模型 & RAG

## 资源

底层后端: 		ollama(大语言模型部署服务)

```sh
github: https://github.com/ollama/ollama
官网url: https://ollama.com/
```

前端: 	maxkb(基于大语言模型和 RAG 的知识库问答系统)

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

ollama 拉取模型

```sh
docker exec -it ollama ollama pull llama3.1
```

ollama 运行模型

```sh
docker exec -it ollama ollama run llama3.1
```

查看所有模型

```sh
docker exec -it ollama ollama list
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

![截图](7af0aa6ed35c286dc0b876370a43acbe.png)

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

MaxKB 不支持直接在 windows 上部署,只能通过 docker 的方式

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

安装成功后，可通过浏览器访问 MaxKB：

```
http://目标服务器 IP 地址:8080

默认登录信息
用户名：admin
默认密码：MaxKB@123..
```
