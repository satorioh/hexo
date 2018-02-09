---
title: 【JS】null和undefined的区别
date: 2018-02-09 15:35:54
categories: JavaScript
tags:
- js
- 前端
permalink: differences-between-null-and-undefined
---
#### 1.获得方式不同
在声明变量但未初始化时，js会自动给变量赋值undefined来完成初始化；而null不会自动赋值，只能由代码生成

#### 2.使用方式不同
undefined表示变量未初始化的状态；而null通常表示一个空对象指针，如果预计变量未来会存放对象，可以初始化为null，或者为了释放内存引用，也可以赋值null<!--more-->

#### 3.转换成数字后的值不同
```javascript
Number(null) //0
Number(undefined) //NaN
```

注意：因为undefined派生自null，所以`undefined == null; //true`
