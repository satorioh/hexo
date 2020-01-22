---
title: Docker初尝试
date: 2020-01-10 09:48:27
categories: Docker
tags:
- docker
permalink: docker-first-try
---
### 方式一：命令行启动（需cd到对应目录下）
##### 1.编写命令行
```shell
docker container run \
-d \
-p 4000:80  \
--rm \
--name payrollCache \
--volume "$PWD/dist":/usr/share/nginx/html \
--volume "$PWD/nginx":/etc/nginx \
nginx
```
<!--more-->

##### 2.修改nginx配置文件
```shell
server {
    listen       80;
    server_name  localhost;
 
    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
 
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
 
    #error_page  404              /404.html;
 
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
 
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    location ~ /api/ {
        proxy_pass   http://172.17.0.1:8097;// docker0接口ip
    }
 
    ...
}
```

### 方式二：简单的Dockerfile
##### 1.编写Dockerfile文件
```dockerfile
FROM nginx:1.17.6
 
LABEL maintainer="robin.wang" date="2019-12-25"
 
LABEL description="payrollCache UI with nginx"
 
COPY ./dist/ /usr/share/nginx/html/
 
COPY ./nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
 
EXPOSE 80
```

##### 2.生成image
```shell
docker image build -t payroll_test:1.0.0 .
```

##### 3.启动容器
```shell
docker container run -d -p 4000:80 --rm --name payrollCache payroll_test:1.0.0
```

### 方式三：使用多步骤构建
##### 1.编写Dockerfile
```dockerfile
FROM node:11.14.0 AS builder
 
COPY package.json package-lock.json /app/
 
WORKDIR /app
 
RUN npm install --registry=https://registry.npm.taobao.org
 
COPY . .
 
RUN npm run build:prod
 
 
FROM nginx:1.17.6
 
LABEL maintainer="robin.wang" date="2019-12-25"
 
LABEL description="payrollCache UI with nginx"
 
COPY --from=builder /app/dist/ /usr/share/nginx/html/
 
COPY --from=builder /app/nginx/default.conf /etc/nginx/conf.d/default.conf
 
EXPOSE 80
```

##### 2.编写.dockerignore文件
```gitignore
.git
node_modules
npm-debug.log
dist
mock
tests
.eslintignore
.eslintrc.js
.gitignore
.travis.yml
jest.config.js
```

##### 3.生成image
```shell
docker image build -t payroll_test:1.0.0 .
```

##### 4.导出image
```shell
docker save -o payroll.tar payroll:1.0.0
```

##### 5.导入image
```shell
docker load -i payroll.tar
```

##### 6.启动容器
```shell
docker container run -d -p 4000:80 --rm --name payrollCache payroll:1.0.0
```

参考文章：
[官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)
[Docker入门教程](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
[Nginx容器教程](https://www.ruanyifeng.com/blog/2018/02/nginx-docker.html)
[Docker容器访问宿主机网络](https://jingsam.github.io/2018/10/16/host-in-docker.html)
[缓存与指令](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache)
