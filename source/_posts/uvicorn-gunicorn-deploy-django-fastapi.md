---
title: Uvicorn + Gunicorn 部署 Django 和 FastAPI
date: 2024-09-21 09:11:07
categories: FastAPI
tags:
- fastapi
- django
- uvicorn
- gunicorn
permalink: uvicorn-gunicorn-deploy-django-fastapi
---
### 一、优势
```
你提到的这个问题非常有意义。Uvicorn 确实可以单独作为 ASGI 服务器运行，而 Gunicorn 通常用于运行 WSGI 应用程序。但是，当你将它们结合在一起使用时，会有一些特定的好处：

1.稳定性：Gunicorn 作为一个长期存在且经过良好测试的项目，它的稳定性和健壮性是有保障的。通过将 Gunicorn 与 Uvicorn 结合使用，可以利用 Gunicorn 的管理能力和稳定性，以及 Uvicorn 的异步性能。

2.进程管理：Gunicorn 提供了一些强大的进程管理功能，例如工作进程的启动、停止和重启。当系统负载增加时，它可以更好地管理资源，而 Uvicorn 单独运行时可能没有这些特性。

3.部署的简化：对于那些已经习惯使用 Gunicorn 来部署 WSGI 应用程序的组织来说，将 Gunicorn 与 Uvicorn 结合使用可以简化迁移到 ASGI 的过程。团队可以利用现有的 Gunicorn 部署、监控和日志记录基础设施。

4.配置灵活性：Gunicorn 提供了许多配置选项和插件，可以方便地自定义应用程序的行为。与 Uvicorn 结合使用时，你可以利用这些灵活性，并允许在同一台机器上同时运行 WSGI 和 ASGI 应用程序。

5.负载均衡：Gunicorn 能够在多个工作进程之间进行负载均衡。这不仅能够更好地利用多核 CPU ，还可以增加系统的可扩展性和冗余性。

总而言之，尽管 Uvicorn 单独使用具有优势，但将 Gunicorn 与 Uvicorn 结合使用可以提供额外的稳定性、灵活性和管理功能。这种组合在具有大量流量和需要高可用性的生产环境中可能尤为有用
```
<!--more-->
### 二、配置
#### 1.安装依赖
```shell
pdm add uvicorn gunicorn
```

#### 2.使用配置文件启动
```shell
import multiprocessing

daemon = True  # 守护进程
bind = '0.0.0.0:8000'  # 绑定ip和端口号
backlog = 512  # 可服务的客户端数量
# chdir = '/home/test/server/bin' #gunicorn要切换到的目的工作目录
timeout = 30  # 超时
worker_class = 'uvicorn.workers.UvicornWorker'  # 使用gevent模式，还可以使用sync 模式，默认的是sync模式
# workers = multiprocessing.cpu_count() * 2 + 1  # 进程数
workers = 1  # 进程数
loglevel = 'info'  # 日志级别，这个日志级别指的是错误日志的级别，而访问日志的级别无法设置
access_log_format = '%(t)s %(p)s %(h)s "%(r)s" %(s)s %(L)s %(b)s %(f)s" "%(a)s"'
accesslog = "./log/gunicorn_access.log"  # 访问日志文件
errorlog = "./log/gunicorn_error.log"  # 错误日志文件
```

#### 3.启动命令
```shell
prod = "doppler run --command='cd ./regulus && gunicorn regulus.asgi:application -c gunicorn.conf.py'"
```

#### 4.设置开机启动服务
```shell
touch /etc/systemd/system/regulus.service
```
```shell
[Unit]
Description=Gunicorn daemon to serve regulus
After=network.target
[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/home/ec2-user/Git/regulus-backend
ExecStart=/home/ec2-user/.local/bin/pdm run prod
[Install]
WantedBy=multi-user.target
```
```shell
chmod +x regulus.service
systemctl daemon-reload
systemctl enable regulus.service
```

#### 5.手动启动/停止服务
```shell
systemctl start nextwebml.service
systemctl stop nextwebml.service
```

### 三、Gunicorn开启https（cloudflare 证书）
1.申请证书：[docs](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca)

2.重命名为key.pem(private)、cert.pem

3.上传到server，修改权限为660

4.修改gunicorn.conf.py 配置如下
```shell
bind = '0.0.0.0:8443'  # 绑定ip和端口号
keyfile = "./cert/key.pem"
certfile = "./cert/cert.pem"
```
5.防火墙开放对应端口

参考：

[如何使用 Uvicorn 托管 Django](https://docs.djangoproject.com/zh-hans/4.2/howto/deployment/asgi/uvicorn/)

[deploy uvicorn using gunicorn](https://www.uvicorn.org/deployment/#gunicorn)

[fastapi deployment](https://fastapi.tiangolo.com/deployment/server-workers/)

[How To Set Up an ASGI Django App with Postgres, Nginx, and Uvicorn on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-asgi-django-app-with-postgres-nginx-and-uvicorn-on-ubuntu-20-04#step-5-creating-systemd-socket-and-service-files-for-gunicorn)

[Gunicorn Settings](https://docs.gunicorn.org/en/stable/settings.html)

[Cloudflare Origin CA certificates](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca)

[Cloudflare Network Ports](https://developers.cloudflare.com/fundamentals/reference/network-ports/)

[Gunicorn running with https](https://www.uvicorn.org/deployment/#running-with-https)
