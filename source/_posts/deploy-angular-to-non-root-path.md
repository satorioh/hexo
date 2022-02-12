---
title: 【Angular】部署到网站非根目录报错解决
date: 2018-05-18 10:12:15
categories: Angular
tags:
- angular
- 前端
permalink: deploy-angular-to-non-root-path
---
### 一、问题描述
需要将angular build后产生的文件，部署到网站的子目录下（比如test.com/dist）来访问，但出现如下图报错：
![deploy .png](../images/deploy%20.png)
<!--more-->
### 二、修复
参考 [angular-cli官方issue](https://github.com/angular/angular-cli/issues/1080)

将build后的index.html中的`<base href="/">`修改为正确的路径：`<base href="/dist/">`，即可解决