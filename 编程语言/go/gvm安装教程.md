[[go]]
1. 安装依赖

```sh
apt install git curl make bison gcc
```

2. 安装 gvm

```sh
bash <<(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
# gvm直接生效
source ~/.gvm/scripts/gvm
# 查看版本
gvm version
```

3. 环境变量

```sh
vim ~/.zshrc
vim ~/.bash_profile
```

添加如下内容：

```sh
# 配置 Golang 软件包镜像
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
# 下载 Golang 的二进制文件或源码压缩包进行安装
export GO_BINARY_BASE_URL=https://mirrors.aliyun.com/golang/
# 根据软件的实际安装情况来选择性加载 gvm
[[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
# 确保 Golang 使用源码编译安装时，不会出错（golang 1.14后需要 ）
export GOROOT_BOOTSTRAP=$GOROOT

```

```sh
# 生效配置
source ~/.zshrc
source ~/.bash_profile

```

4. 使用指定版本的 go

```sh
gvm use go1.15.7
```

5. 设置默认版本的 go

```sh
gvm use go1.16.15 --default
```
6. 常见的 gvm 指令
```bash
gvm list              # 已装的 Go 版本
gvm listall           # 远端所有可装版本
gvm install go1.22.5  # 从源码编译安装指定版本(首次较慢)
gvm install go        # 装最新稳定版
gvm use go1.22.5      # 当前 shell 切到该版本(需先 install)
gvm use system        # 切回系统 Go
gvm use go1.22.5 --default   # 设为默认版本
gvm remove go1.22.5   # 卸载某版本
gvm alias default go1.22.5   # 设默认(等价于 --default)
gvm env               # 查看当前 GVM 环境变量
gvm which             # 当前 go 实际路径
gvm go_path           # 当前 GOPATH
gvm go bin            # 当前 GOBIN
gvm go src            # 当前 GOPATH/src
gvm go pkg            # 当前 GOMODCACHE
gvm go doc            # 当前 GOPATH/pkg
gvm help              # 全部命令
```