---
title: 使用docker compose安装LGP日志系统
date: 2021-08-21 16:44:12
categories: Docker
tags:
- docker
- lgp
permalink: use-docker-compose-install-lgp
---
环境：
> CentOS: 7
> 
> Docker: 20.10.8
> 
> Docker Compose: 1.29.2
> 
> Loki: 2.3.0
> 
> Promtail: 2.3.0
> 
> Grafana: latest
<!--more-->

#### 一、[安装docker compose](https://docs.docker.com/compose/install/)
```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

#### 二、获取LGP配置文件
##### 1.docker-compose.yaml
```yaml
version: "3"

networks:
  loki:

services:
  loki:
    image: grafana/loki:2.3.0
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - loki

  promtail:
    image: grafana/promtail:2.3.0
    volumes:
      - ~/loki/config.yml:/etc/promtail/config.yml
      - /var/log:/var/log
      - ~/.pm2/logs:/pm2/logs
    command: -config.file=/etc/promtail/config.yml
    networks:
      - loki

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - ~/loki/grafana.ini:/etc/grafana/grafana.ini
    networks:
      - loki
```

##### 2.Promtail配置单独挂载
```yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log

- job_name: pm2
  static_configs:
  - targets:
      - localhost
    labels:
      job: pm2logs
      __path__: /pm2/logs/*.log
```

##### 3.设置grafana免登录
```shell
[auth.anonymous]
enabled = true
org_role = Admin
```

#### 三、使用docker-compose启动
##### 1.启动
```shell
sudo docker-compose up -d # -d：后台运行
```

##### 2.查看
```shell
sudo docker-compose ps
```

##### 3.停止并删除
```shell
sudo docker-compose stop
sudo docker-compose rm
```

参考文章：

[How to use custom ini file for Grafana with Docker?](https://community.grafana.com/t/how-to-use-custom-ini-file-for-grafana-with-docker/45492)

[Is it possible to completely disable auth?](https://community.grafana.com/t/is-it-possible-to-completely-disable-auth/17306)
