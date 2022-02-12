---
title: Hexo 相关问题汇总
date: 2022-02-12 18:13:48
categories: Hexo
tags:
permalink: hexo-problem
---
##### 1.TypeError [ERR_INVALID_ARG_TYPE]: The "mode" argument must be integer. Received an instance of Object
解决：hexo 不支持最新 node14+，降低node版本到12

##### 2.git@github.com: Permission denied (publickey).fatal: Could not read from remote repository.

解决：将.ssh目录下的id_rsa.pub加到Github ssh key中

命令：将公钥复制到剪贴板：`pbcopy < ~/.ssh/id_rsa.pub`
<!--more-->
参考文章：

[node14+版本下hexo部署失败](https://evestorm.github.io/posts/430/)

[处理 git@github.com: Permission denied (publickey)](https://cloud.tencent.com/developer/article/1775620)
