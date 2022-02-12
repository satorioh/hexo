---
title: 修改docker0默认ip
date: 2020-04-09 11:04:09
categories: Docker
tags:
- docker
permalink: docker0-ip-change
---
##### 1.查看当前docker0接口ip
```shell
ifconfig
```
<!--more-->

##### 2.切换root用户
```shell
sudo su -
```

##### 3.修改 /etc/docker/daemon.json (没有需创建)
```shell
{
    "bip":"169.254.123.1/24"
}
```

##### 4.重启docker服务
```shell
systemctl restart docker
```

参考：
[Docker 修改Docker0网桥默认网段](https://cloud.tencent.com/developer/article/1470937)
