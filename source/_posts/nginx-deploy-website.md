---
title: CentOS下Nginx简易部署网站
date: 2020-03-26 13:12:48
categories: Nginx
tags:
- nginx
permalink: nginx-deploy-website
---
#### 1.安装
```
yum install -y nginx // 可以root安装，CentOS
```
<!--more-->

#### 2.修改配置文件
```
// 配置文件：/etc/nginx/conf.d/default.conf

server {
	listen   4000; // 修改默认监听端口
}

location ~ /api/  {
	proxy_pass http://ip:port // 修改proxy配置，指向后端
}
```

#### 3.修改运行时的用户(解决403 forbidden)
```
// 配置文件：/etc/nginx/nginx.conf

user root;
```

#### 4.添加网站静态资源文件
```
cp -r * /user/share/nginx/html
```

#### 5.启动nginx
```
systemctl start nginx
```

#### 6.添加nginx开机启动
```
systemctl enable nginx
```
