---
title: 【CSS】scrollbar样式自定义
date: 2019-04-11 10:11:50
categories: CSS
tags:
- css
- 前端
permalink: css-scrollbar-style-change
---
```css
._scrollbar::-webkit-scrollbar-track-piece {
	background: #F0F0F0;
}
._scrollbar::-webkit-scrollbar {
	width: 10px;
	height: 10px;
}
._scrollbar::-webkit-scrollbar-thumb {
	background: #81C8FF;
	background-clip:padding-box;
	min-height:28px;
	border-radius: 5px;
}
._scrollbar::-webkit-scrollbar-thumb:hover {
	background-color:#bbb;
	cursor: pointer;
}
```
