---
title: 安装docker及常用指令
date: 2020-01-25 11:01:35
categories: Docker
tags:
- docker
permalink: docker-installation
---
### 一、安装
##### 1.安装repo工具
```shell
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
```
<!--more-->

##### 2.安装repo
```shell
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

##### 3.安装
```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

##### 4.加入用户组
```shell
sudo usermod -aG docker $USER
```

##### 5.启动服务
```shell
sudo systemctl start docker
```

### 二、常用指令
##### 1.列出本机的所有 image 文件
```shell
docker image ls
```

##### 2.删除 image 文件
```shell
docker image rm [imageName]
```

##### 3.下载image
```shell
docker pull [imageName]
```

##### 4.运行image
```shell
docker run [imageName]
```

##### 5.列出本机正在运行的容器
```shell
docker container ls
```

##### 6.列出本机所有容器，包括终止运行的容器
```shell
docker container ls --all
```

##### 7.删除容器
```shell
docker container rm [containerId]
```

##### 8.生成image
```shell
docker image build -t payroll_test:1.0.0 .
```

##### 9.启动容器
```shell
docker container run -d -p 4000:80 --rm --name payrollCache payroll_test:1.0.0
```

##### 10.导出image
```shell
docker save -o payroll.tar payroll:1.0.0
```

##### 11.导入image
```shell
docker load -i payroll.tar
```

##### 12.进入容器内部
```shell
docker exec -it [containerId] bash
```

##### 13.下载容器内文件
```shell
docker cp [containerId]:/[source path] [dest path]
```

参考文章：
[官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)
