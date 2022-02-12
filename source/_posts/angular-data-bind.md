---
title: 【Angular】数据绑定
date: 2018-04-25 14:04:55
categories: Angular
tags:
- angular
- 前端
permalink: angular-data-bind
---
### 一、事件绑定
`<button (click)= "onButtonClick($event)"></button>`

### 二、DOM属性绑定（插值表达式）
`<img [src]="imgUrl">`
`<img src="{{imgUrl}}">`
<!--more-->
### 三、HTML属性绑定
#### 1.基本HTML属性绑定
`<td [attr.colspan]="tableColspan"></td>`

#### 2.CSS类绑定

| 动态效果       | 代码                                                               |
|:-------------|:-------------------------------------------------------------------|
| 全部替换       | `<div class="aaa bbb" [class]="someVarible">`                      |
| 增加/删除一个类 | `<div [class.ccc]="isDev" >`                                       |
| 增加/删除多个类    | `<div [ngClass]="obj" > ` //obj = {aaa:true, bbb:false, ccc:false} |

#### 3.样式绑定

| 动态效果    | 代码                                                                                              |
|:-----------|:-------------------------------------------------------------------------------------------------|
| 增加一个样式 | `<button [style.color]="isDev? 'red':'green'" >` ; `<button [style.font-size.em]="isDev? 3:1" >` |
| 增加多个样式 | `<div [ngStyle]="obj">` //obj={color:'red',background:'yellow'}                                  |

### 四、双向绑定（适合表单元素）
`<input [(ngModel)]="name"> {{name}}`