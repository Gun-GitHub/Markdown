## 使用 NVM 创建环境

```shell
nvm install 22
nvm use 22
# 检查当前 node
node -v
```

## 安装 openclaw

```shell
# 这个是英文版
npm install -g openclaw@latest --registry=https://mirrors.tuna.tsinghua.edu.cn/npm/
# 这个是汉化版本
npm install -g openclaw-cn@latest --registry=https://mirrors.tuna.tsinghua.edu.cn/npm/
# 检查是否有 openclaw 指令
openclaw --help
```

## 交互式引导配置

```shell
# 交互式引导配置（推荐新手）用这个就行,下面的不用
openclaw onboard

# 打开控制面板
openclaw dashboard

# 首次安装后初始化配置
openclaw setup
```

