---
title: 【Vue】使用Cordova打包Vue webapp
date: 2018-06-07 15:44:34
categories: VUE
tags:
- vue
- 前端
permalink: vue-packed-with-cordova
---
#### 一、安装并创建vue project
```shell
npm install -g vue-cli
vue init webpack my-vue
cd my-vue
npm install
npm run dev
```
到localhost:8080下访问页面正常，则OK
<!--more-->

#### 二、安装并创建cordova project
```shell
npm install -g cordova
```
如果此处报错，可以先卸载（`npm uninstall -g cordova`）再尝试安装

创建project，建议和之前的my-vue放在同一个目录下，方便后续写路径
```shell
cordova create my-cordova
```

#### 三、添加平台并运行
```shell
cordova platform add android
```
运行cordova:
```shell
cordova run android
```
如果此时报错，可参考我之前写的 [Ionic环境搭建及基本命令](https://roubin.me/ionic-environment-build-and-basic-command/)中的“二、环境变量设置”，把jdk和android studio安装配置好

生成apk文件：
```shell
cordova build android
```

#### 四、修改my-vue中的build配置
1.修改my-vue下的index.html文件，增加3个meta和1个cordova.js，如下
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *; img-src * data: content:;">
    <meta name="format-detection" content="telephone=no">
    <meta name="msapplication-tap-highlight" content="no">
    <title>vue-travel</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
    <script type="text/javascript" src="cordova.js"></script>
  </body>
</html>

```

2.修改config/index.js文件的build部分，指向my-cordova的www目录，如下：
```js
build: {
    // Template for index.html
    index: path.resolve(__dirname, '../../my-cordova/www/index.html'),

    // Paths
    assetsRoot: path.resolve(__dirname, '../../my-cordova/www'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '',
```
这样vue build后的文件会直接到my-cordova/www下

#### 五、生成apk
切换到my-cordova，运行`cordova build android`，即可生成apk文件
之后每次重新编译，需要手动删除之前www下生成的文件

参考链接：

[cordova+vue 项目打包成Android（apk）应用](https://segmentfault.com/a/1190000013159076)

[教你用Cordova打包Vue项目](https://www.jianshu.com/p/25d797b983cd)

[Vue 2.0 + cordova 构建Android应用](https://segmentfault.com/a/1190000008281748)

[cordova + vue-cli构建跨平台应用](https://www.jianshu.com/p/d25b2d6a4e04)



