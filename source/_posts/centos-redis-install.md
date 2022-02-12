---
title: Redis 6.x在MacOS/CentOS 7上的安装与配置
date: 2021-08-20 14:28:46
categories: Linux
tags:
- centos
- redis
permalink: centos-redis-install
---
环境：
> redis:6.2.5
> 
> macOS:10.15.7
> 
> CentOS:7
### 一、MacOS下安装与配置
#### 1.安装
```shell
brew install redis
```
<!--more-->

#### 2.修改配置
```shell
# 配置文件：/usr/local/etc/redis.conf

# 修改端口
Port 6379

# 密码设置
requirepass 123456
```

#### 3.常用命令
##### a.启动redis服务
```shell
brew services start redis
```

##### b.卸载redis
```shell
brew uninstall redis rm ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```


### 二、CentOS 7下安装与配置
#### 1.升级gcc
```shell
# 查看版本，redis6(redis 5不需要)需要较高版本的gcc编译
gcc -v

# 升级
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile  ## gcc版本永久生效
```

#### 2.安装
```shell
# 如果要安装最新的redis，需要安装Remi的软件源，官网地址：http://rpms.famillecollet.com/
yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

# 然后可以使用下面的命令安装最新版本的redis
yum --enablerepo=remi install -y redis
```

#### 3.配置
```shell
# 配置文件 /etc/redis.conf

# 注释掉绑定ip，允许远程连接
bind 127.0.0.1 # 注释掉这句

# 允许后台运行
daemonize yes

# 关闭保护模式，否则外部ip无法连接
protected-mode no

# 修改端口
Port 6379

# 密码设置
requirepass 123456
```

#### 4.常用命令
```shell
# 启动redis
service redis start

# 停止redis
service redis stop

# 查看redis运行状态
service redis status

#设置redis为开机自动启动
chkconfig redis on
```

#### 5.配置防火墙
```shell
# 切换root
sudo su -

# 查看当前防火墙开启的端口
firewall-cmd --list-all

# 永久开启指定端口
firewall-cmd --permanent --add-port=6379/tcp

# 重启防火墙
firewall-cmd --reload
```

参考文章：

[centos7 yum install redis](https://www.cnblogs.com/autohome7390/p/6433956.html)

[CentOS7 linux下yum安装redis以及使用](https://cloud.tencent.com/developer/article/1653709)

[在Mac上安装redis](https://cloud.tencent.com/developer/article/1606701)
