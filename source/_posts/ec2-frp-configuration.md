---
title: aws ec2 frp 配置
date: 2024-09-21 08:57:21
categories: Linux
tags:
- frp
- aws
- ec2
permalink: ec2-frp-configuration
---
#### 1.客户端和服务端安装Go：[docs](https://go.dev/doc/install#requirements)

#### 2.服务端ec2上把Go加入Path
```shell
echo 'export PATH="$PATH:/usr/local/go/bin"' >> ~/.bashrc
echo 'export PATH="$PATH:/usr/local/go/bin"' >> ~/.bash_profile
```
<!--more-->

#### 3.下载并安装c/s程序：[docs](https://github.com/fatedier/frp/releases)

#### 4.修改配置文件：[示例](https://gofrp.org/zh-cn/docs/examples/)
服务端配置：
```shell
bindPort = 7000
vhostHTTPPort = 8080
```

客户端配置：
```shell
serverAddr = "xxx.amazonaws.com"
serverPort = 7000

[[proxies]]
name = "test-tcp"
type = "http"
localIP = "127.0.0.1"
localPort = 8000
customDomains = ["xxx.amazonaws.com"]
```

#### 5.访问`xxx.amazonaws.com:8080`可到达本地 8000

#### 6.配置systemd启动：[frp使用systemd](https://gofrp.org/zh-cn/docs/setup/systemd/)
```shell
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /home/ec2-user/frp/frps -c /home/ec2-user/frp/frps.toml

[Install]
WantedBy = multi-user.target
```

#### 7.systemd常用命令
```shell
# 启动frp
sudo systemctl start frps

# 停止frp
sudo systemctl stop frps

# 重启frp
sudo systemctl restart frps

# 查看frp状态
sudo systemctl status frps

```
