---
title: Mac M1 安装 NVM
date: 2022-02-12 20:14:33
categories: Mac
tags:
- nvm
permalink: mac-m1-nvm
---
##### 1.下载安装nvm
```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
或者
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

##### 2.在.zshrc中配置环境变量
```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```
<!--more-->

##### 3.下载低于15版本的Node
```shell
node -p process.arch  #查看当前架构
arch -x86_64 zsh      #更换为64位架构
nvm install v12.22.10 #下载低于15版本的Node
exit                  #退出zsh
nvm use v12.22.10     #使用低于15版本的Node
```
参考文章：


[对于M1芯片的Mac在安装NVM，并用nvm下载不同的node版本的时候遇到的坑](https://juejin.cn/post/7002566911456182303)
