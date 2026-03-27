## 下载 NVM

```shell
wget https://github.com/nvm-sh/nvm/archive/refs/tags/v0.39.1.tar.gz
```

## 解压安装包

```
tar -zxvf v0.39.1.tar.gz
```

## 移动到指定目录

放在你常用路径,哪里都行

```shell
mv nvm-0.39.1  /home/zhoujun/snap/
```

## 添加用户环变量

```shell
cat >> ~/.bashrc << EOF

#========================= nvm envs =============================
export NVM_DIR="/home/zhoujun/snap/nvm-0.39.1"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
#========================= nvm envs =============================

EOF
```

## 重载环境变量

```shell
source ~/.bashrc
```

## 测试指令

```shell
zhoujun@ubuntu:~/下载$ nvm -v
0.40.1
```

## 常用指令

```shell
nvm install 14.13.2
 
#nvm常用命令
nvm uninstall 14.13.2     # 移除 node 14.13.2
nvm use 14.13.2           # 使用 node 14.13.2
nvm ls                    # 查看目前已安装的 node 及当前所使用的 node
nvm ls-remote             # 查看目前线上所能安装的所有 node 版本
nvm alias default 14.13.2 # 使用 14.13.2 作为预设使用的 node 版本
```

