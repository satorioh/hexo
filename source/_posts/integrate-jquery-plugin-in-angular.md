---
title: 【Angular】在Angular 5中引入jQuery插件
date: 2018-04-27 14:26:13
categories: Angular
tags:
- angular
- 前端
permalink: integrate-jquery-plugin-in-angular
---
#### 1.安装jQuery
`npm install --save jquery`

#### 2.安装jQuery定义文件
`npm install @types/jquery --save`
<!--more-->
#### 3.安装jQuery插件（以daterangepicker为例）
`npm install bootstrap-daterangepicker --save`

#### 4.在ng目录下.angular-cli.json的"styles"中引入daterangepicker.css
`"styles": [daterangepicker.css]`

#### 5.在对应的组件中导入jQuery和daterangepicker
```typescript
import * as $ from 'jquery';
import 'bootstrap-daterangepicker';
```

#### 6.在typings.d.ts中为插件添加定义
```typescript
interface JQuery {
   daterangepicker(options?: any, callback?: Function) : any;
}
```

参考链接：

[Typescript — Integrate jQuery Plugin in your Project](https://medium.com/@NetanelBasal/typescript-integrate-jquery-plugin-in-your-project-e28c6887d8dc)