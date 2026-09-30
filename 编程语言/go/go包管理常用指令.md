#### Go Mod 常用命令速查表

|命令|作用|说明|
|---|---|---|
|`go mod init <module>`|初始化模块|生成 `go.mod` 文件|
|`go mod download`|下载依赖|下载到 `$GOPATH/pkg/mod`|
|`go mod tidy`|整理依赖|添加缺失、删除多余，最常用|
|`go mod verify`|校验依赖|检查依赖是否被篡改|
|`go mod graph`|查看依赖图|输出模块依赖关系|
|`go mod why <pkg>`|查看依赖原因|解释某包为何被引入|
|`go mod edit`|编辑 go.mod|命令行修改 go.mod|
|`go mod vendor`|生成 vendor 目录|复制依赖到 `vendor/`|
|`go clean -modcache`|清理缓存|删除所有模块缓存|
#### go mod edit 参数

|参数|作用|示例|
|---|---|---|
|`-require=path@ver`|添加依赖|`-require=github.com/pkg/errors@v0.9.1`|
|`-droprequire=path`|删除依赖|`-droprequire=github.com/pkg/errors`|
|`-replace=old=new`|替换依赖|`-replace=github.com/old=new@v1.0.0`|
|`-dropreplace=old`|删除替换|`-dropreplace=github.com/old`|
|`-module=path`|修改模块路径|`-module=github.com/new/name`|
|`-go=1.21`|修改 Go 版本|`-go=1.21`|
|`-json`|JSON 格式输出|`-json`|

#### go get 相关

|命令|作用|
|---|---|
|`go get <pkg>`|添加依赖（最新版）|
|`go get <pkg>@<ver>`|添加指定版本|
|`go get <pkg>@latest`|升级到最新版|
|`go get -u ./...`|升级所有直接依赖|
|`go get -u=patch ./...`|只升级补丁版本|

#### go list 相关

|命令|作用|
|---|---|
|`go list -m all`|列出所有依赖及版本|
|`go list -m -u all`|列出可升级的依赖|
|`go list -m -versions <pkg>`|列出某模块所有可用版本|

#### 核心文件

|文件|作用|
|---|---|
|`go.mod`|声明模块路径、Go 版本、依赖|
|`go.sum`|依赖哈希校验值|
|`vendor/`|依赖副本（离线构建）|

#### 典型工作流

|步骤|命令|说明|
|---|---|---|
|1|`go mod init github.com/example/app`|初始化|
|2|编写代码，import 需要的包|—|
|3|`go mod tidy`|自动整理依赖|
|4|`go build ./...`|构建|
|5|`go mod verify`|校验（可选）|

#### 版本后缀说明

|后缀|含义|
|---|---|
|`@latest`|最新版本|
|`@v1.2.3`|指定版本|
|`@v1.2`|该次版本最新|
|`@patch`|最新补丁版本|
|`@none`|移除依赖|
|`@master`|分支名|
|`@<commit-hash>`|指定提交|

#### go mod 环境变量
可以通过 go env 命令来进行查看
```shell
zhoujun@ubuntu:~/snap/gvm/pkgsets/go1.27.0/global/pkg/mod$ go env
AR='ar'
CC='gcc'
CGO_CFLAGS='-O2 -g'
CGO_CPPFLAGS=''
CGO_CXXFLAGS='-O2 -g'
CGO_ENABLED='1'
CGO_FFLAGS='-O2 -g'
CGO_LDFLAGS='-O2 -g'
CXX='g++'
GCCGO='gccgo'
GO111MODULE='on'                                     # 很重要
GOAMD64='v1'
GOARCH='amd64'
GOAUTH='netrc'
GOBIN='/home/zhoujun/snap/gvm/pkgsets/go1.27.0/global/bin'
GOCACHE='/home/zhoujun/.cache/go-build'
GOCACHEPROG=''
GODEBUG=''
GOENV='/home/zhoujun/.config/go/env'
GOEXE=''
GOEXPERIMENT=''
GOFIPS140='off'
GOFLAGS=''
GOGCCFLAGS='-fPIC -m64 -pthread -Wl,--no-gc-sections -fmessage-length=0 -ffile-prefix-map=/tmp/go-build1989038984=/tmp/go-build -gno-record-gcc-switches'
GOHOSTARCH='amd64'
GOHOSTOS='linux'
GOINSECURE=''
GOMOD='/dev/null'
GOMODCACHE='/home/zhoujun/snap/gvm/pkgsets/go1.27.0/global/pkg/mod'
GONOPROXY=''                                          # 很重要
GONOSUMDB=''                                         # 很重要
GOOS='linux'
GOPACKAGESDRIVER=''
GOPATH='/home/zhoujun/snap/gvm/pkgsets/go1.27.0/global'
GOPRIVATE=''
GOPROXY='https://goproxy.cn,direct'      # 很重要
GOROOT='/home/zhoujun/snap/gvm/gos/go1.27.0'
GOSUMDB='sum.golang.org'                   # 很重要
GOTELEMETRY='local'
GOTELEMETRYDIR='/home/zhoujun/.config/go/telemetry'
GOTMPDIR=''
GOTOOLCHAIN='auto'
GOTOOLDIR='/home/zhoujun/snap/gvm/gos/go1.27.0/pkg/tool/linux_amd64'
GOVCS=''
GOVERSION='go1.27.0'
GOWORK=''
PKG_CONFIG='pkg-config'
```

#### GO111MODULE
Go 语言提供了 GO111MODULE 这个环境变量来作为 Go modules 的开关, 其允许设置以下参数:
- auto: 只要项目包含了 go.mod 文件的话器用 Go modules
- on: 启用 Go modules, 推荐使用
- off: 禁用 Go modules, 不推荐使用
可以通过以下指令来设置:
```go
go env -w GO111MODULE=on
```
#### GOPROXY
这个环境遍变量主要是用于设置 Go 模块代理(go module proxy), 其作用是用于使 Go 再后续拉取模块版本时直接通过镜像站点来快速拉取
默认值是:
```go
GOPROXY='https://goproxy.cn,direct'
```
- 阿里云
    - https://mirrors.aliyun.com/goproxy/
- 七牛云
    - https://goproxy.cn,direct
#### GOSUMDB
校验第三方库的一个网站
#### GOPRIVATE
公司或者自写的第三方库
写法
```go
go env -w GOPRIVATE="git.example.com, github.com/zceld/zinx"
go env -w GOPRIVATE="*.example.com" // 通配符
```
不会去 GOPROXY, GOSUMDB