---
title: 【Angular】Angular 5 整合 AdminLTE 2.4.3
date: 2018-04-20 08:33:44
categories: Angular
tags:
- angular
- 前端
permalink: angular-integrated-with-adminlte
---
1.下载 [Admin LTE](https://github.com/almasaeed2010/AdminLTE/releases)，并解压到本地

2.将解压所得的三个文件夹：bower_components、dist、plugins，拷贝至ng目录的src/assets下
<!--more-->
3.用编辑器打开admin LTE解压文件夹内的index2.html

4.拷贝引用的css内容至ng目录下angular-cli.json的styles内，同时修改为正确的路径，如下：
```json
"styles": [
        "styles.css",
        "assets/bower_components/bootstrap/dist/css/bootstrap.min.css",
        "assets/bower_components/font-awesome/css/font-awesome.min.css",
        "assets/bower_components/Ionicons/css/ionicons.min.css",
        "assets/bower_components/jvectormap/jquery-jvectormap.css",
        "assets/dist/css/AdminLTE.min.css",
        "assets/dist/css/skins/_all-skins.min.css"
      ],
```
5.拷贝index2.html内的js引用，至angular-cli.json的scripts内（可根据用到的功能自行增删），同时修改为正确的路径，如下：
```json
"scripts": [
        "assets/bower_components/jquery/dist/jquery.min.js",
        "assets/bower_components/bootstrap/dist/js/bootstrap.min.js",
        "assets/bower_components/fastclick/lib/fastclick.js",
        "assets/dist/js/adminlte.min.js",
        "assets/bower_components/jquery-sparkline/dist/jquery.sparkline.min.js",
        "assets/plugins/jvectormap/jquery-jvectormap-1.2.2.min.js",
        "assets/plugins/jvectormap/jquery-jvectormap-world-mill-en.js",
        "assets/bower_components/jquery-slimscroll/jquery.slimscroll.min.js",
        "assets/bower_components/chart.js/Chart.js"
      ],
```

6.【可选】拷贝index2.html内的google fonts引用，至ng目录src/index.html的head标签内

7.为src/index.html的body标签添加class（此处依据index2.html的body拥有的class），如下：
```html
<body class="hold-transition skin-blue sidebar-mini">
  <app-root></app-root>
</body>
```

8.拷贝index2.html中body下div.wrapper全部内容，至ng目录下src/app/app.component.html中

9.运行ng serve --open

10.依据页面情况，调整缺失图片的相关路径即可

11.创建各部分组件，将对应内容分配到组件中，如下（app.component.html）
```html
<div class="wrapper">

  <!-- Main Header -->
  <app-header></app-header>
  <!-- Left side column. contains the logo and sidebar -->
  <app-menu></app-menu>

  <!-- Content Wrapper. Contains page content -->
  <app-content></app-content>
  <!-- /.content-wrapper -->

  <!-- Main Footer -->
  <app-footer></app-footer>

  <!-- Control Sidebar -->
  <app-sidebar></app-sidebar>
</div>

```

参考链接：

[ANGULAR 4 Admin LTE Theme Integration Advanced](https://www.youtube.com/watch?v=4YRVuRN5k04)
