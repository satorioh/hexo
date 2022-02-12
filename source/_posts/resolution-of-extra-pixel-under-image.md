---
title: 【CSS】image下方多余像素问题解决
date: 2017-05-04 14:15:20
categories: CSS
tags:
- css
- 前端
permalink: resolution-of-extra-pixel-under-image
---
**问题代码：**
```html
<!doctype html>
<html>
 <head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
	p {
		border:1px solid red;
		background:#eee;
	}
	#img2 {
		width:200px;
		height:200px;
	}
  </style>
 </head>
 <body>
  <p><img src="1.jpg" id="img2"></p>
 </body>
</html>
```
<!--more-->
**方法1：**设置image的display为block
```css
p {
		border:1px solid red;
		background:#eee;
	}
#img2 {
		display:block;
		width:200px;
		height:200px;
	}
```

**方法2：**修改image的vertical-align为非默认值（middle/top/bottom）
```css
p {
		border:1px solid red;
		background:#eee;
	}
#img2 {
		width:200px;
		height:200px;
		vertical-align:middle;
	}
```

**方法3：**设置p的line-height为0
```css
p {
		border:1px solid red;
		background:#eee;
		line-height:0;
	}
#img2 {
		width:200px;
		height:200px;
	}
```

**方法4：**设置p的font-size为0
```css
p {
		border:1px solid red;
		background:#eee;
		font-size:0;
	}
#img2 {
		width:200px;
		height:200px;
	}
```