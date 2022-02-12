---
title: CentOS防火墙开启端口
date: 2020-04-02 11:17:16
categories: Linux
tags:
- centos
- firewall
permalink: centos-firewall-open-port
---
##### 1.切换root
```shell
sudo su -
```
<!--more-->

##### 2.查看当前防火墙开启的端口
```shell
firewall-cmd --list-all
```

##### 3.永久开启指定端口
```shell
firewall-cmd --permanent --add-port=8099/tcp
```

##### 4.重启防火墙
```shell
firewall-cmd --reload
```

