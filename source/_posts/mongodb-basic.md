---
title: MongoDB 基础
date: 2021-06-28 11:15:18
categories: MongoDB
tags:
- mongodb
permalink: mongodb-basic
---
## 一、Mac下安装与运行
### 方式一：使用Homebrew安装([参考](https://www.mongodb.com/docs/v6.0/tutorial/install-mongodb-on-os-x/))
```shell
brew tap mongodb/brew
brew update
brew install mongodb-community@x.x

mongod --config /opt/homebrew/etc/mongod.conf #直接启动
brew services start mongodb/brew/mongodb-community #作为服务自动启动
```

### 方式二：手动下载安装
#### 1.下载
[官方下载](https://www.mongodb.com/try/download/community)， 解压下载的文件，无需安装，直接拷贝到 /usr/local 目录，然后重命名为 mongodb

#### 2.设置PATH
如果要使用 MongoDB 的命令行，需要添加一个环境变量，找到 .zshrc 文件，添加以下代码：
```shell
export PATH=/usr/local/mongodb/bin:$PATH
```
生效：
```shell
source ~/.zshrc
```
<!--more-->
#### 3.设置数据目录
```shell
sudo mkdir -p /usr/local/var/mongodb //数据存放路径

sudo mkdir -p /usr/local/var/log/mongodb //日志文件路径
```

#### 4.设置权限
确保当前用户对以上两个目录有读写的权限
```shell
sudo chown `你的用户名` /usr/local/var/mongodb
sudo chown `你的用户名` /usr/local/var/log/mongodb
```

#### 5.启动 MongoDB
```shell
mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork

--dbpath 设置数据存放目录
--logpath 设置日志存放目录
--fork 在后台运行
```

#### 6.查看服务是否启动
```shell
ps aux | grep -v grep | grep mongod
```

#### 7.开启命令行终端
```shell
mongo
```

#### 8.结束服务
```shell
mongo

> use admin;
> db.shutdownServer();
```

## 二、Node使用 mongoose 操作 MongoDB
#### 1.连接db
```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true, useUnifiedTopology: true});
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function (callback) {
    console.log("DB Connected")
});
```

#### 2.使用 Schema 和 model 建立 document
```javascript
const loginSchema = new mongoose.Schema({
    username:String,
    password:String
});
const login = db.model("login",loginSchema,"login");

async function createUser(username,password){
    const user = new login({username, password});

    await user.save().then(function (err) {
        if (err) {
            console.log(err);
            return;
        }
        console.log("create user done");

    });
}

createUser('bill', '1234');
```

#### 2.搜索
```javascript
const query = {id:'xxx'};
const result = await blogList.find(query);
```

#### 3.删除
```javascript
const query = {id:'xxx'};
await blogList.remove(query);
```


参考文章：

[Mac OSX 平台 MongoDB 的安装及管理](https://cloud.tencent.com/developer/article/1770288)

[Install MongoDB Community Edition on macOS](https://www.mongodb.com/docs/v6.0/tutorial/install-mongodb-on-os-x/)
