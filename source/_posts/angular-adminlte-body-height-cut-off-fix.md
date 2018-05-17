---
title: 【Angular】AdminLTE body高度异常问题解决
date: 2018-05-17 15:04:21
categories: Angular
tags:
- angular
- 前端
permalink: angular-adminlte-body-height-cut-off-fix
---
### 一、问题描述
如下图，从登录页进入主页后，body高度异常，此issue应该不限于angular与adminLTE的组合
![cutoff .PNG](../images/cutoff%20.PNG)
<!--more-->
**版本：**
- Angular : 5
- AdminLTE: 2.4.3

### 二、解决
参考 [AdminLTE官方issue](https://github.com/almasaeed2010/AdminLTE/issues/1033)

在对应的组件类中添加如下代码，即可修复
```typescript
declare var $:any;

ngOnInit() {
    $('body').layout('fix');
  }
```

对应的模板：
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
  <!---->
  <!-- /.control-sidebar -->
  <!-- Add the sidebar's background. This div must be placed
  immediately after the control sidebar -->
</div>
<!-- ./wrapper -->
```