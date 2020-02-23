---
title: Git、npm代理设置
date: 2020-01-25 11:25:25
categories: Git
tags:
- git
- npm
permalink: git-npm-proxy-settings
---
### 一、Git代理设置
##### 1.查看
```shell
git config -l
```
<!--more-->

##### 2.设置
```shell
git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

##### 3.取消设置
```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 二、npm代理设置
##### 1.查看
```shell
npm config list
```

##### 2.设置
```shell
npm config set proxy=http://localhost:1080
npm config set https-proxy=http://localhost:1080
```

##### 3.取消设置
```shell
npm config delete proxy
npm config delete https-proxy
```

### 三、使用nrm
```shell
//1.先安装nrm工具
npm install -g nrm

//2.查看当前可用的镜像源
nrm ls

//3.切换npm源
nrm use 镜像源名称
```
