---
title: pm2基础
date: 2021-08-20 10:08:41
categories: Node
tags:
- pm2
permalink: pm2-basic
---
### 一、命令
#### 1.设置开机启动
```shell
pm2 startup
```

#### 2.关闭开机启动
```shell
pm2 unstartup systemd
```
<!--more-->

#### 3.无中断式重启
```shell
pm2 reload app
```

#### 4.清空所有应用日志
```shell
pm2 flush
```

#### 5.以cluster模式启动
```shell
pm2 start index.js -i n/max // max: 按cpu数量自动分配
```

### 二、配置([Ecosystem选项参考](https://www.bookstack.cn/read/pm2-runtime/22.md))
#### 1.以json格式输出日志
```shell
--log-type=json
````

#### 2.cluster模式下日志合并
```shell
--merge-logs
````

#### 3.以server所在时区显示timestamp([遵循moment.js](https://momentjs.com/docs/#/parsing/string-format/))
```shell
--log-date-format 'YYYY-MM-DD HH:mm:ss'
```

### 三、[pm2-logrotate](https://github.com/keymetrics/pm2-logrotate)
#### 1.安装
```shell
pm2 install pm2-logrotate
```

#### 2.日志位置：
```shell
~/.pm2/logs/
```

#### 3.设置
```shell
pm2 set pm2-logrotate:option value
```

参考文章：

[PM2教程](https://www.bookstack.cn/read/pm2-runtime/0.md)

[Ecosystem选项参考](https://www.bookstack.cn/read/pm2-runtime/22.md)

[pm2-logrotate](https://github.com/keymetrics/pm2-logrotate)
