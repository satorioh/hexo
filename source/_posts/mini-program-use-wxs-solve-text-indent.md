---
title: 【微信小程序】使用wxs解决文本替换与首行缩进
date: 2019-04-01 11:12:34
categories: 小程序
tags:
- 微信小程序
- 前端
permalink: mini-program-use-wxs-solve-text-indent
---
#### 一段文本缩进
text标签在小程序中默认为内联标签，如果单独设置 text-indent是无效的，需设置为块级/行内块元素
```css
display: inline-block;
text-indent: 58rpx;
```
<!--more-->

#### 多段文本换行与缩进
```html
<wxs src="../../utils/filter.wxs" module="tools"></wxs>
<text decode="{{true}}">{{tools.format(多段文本的内容)}}</text>
//decode译码解码
```

filter.wxs:
```javascript
var format = function (text){
    if(!text) return
    var reg = getRegExp('\\\\n', 'g')
    return text.replace(reg, '\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;')
}

module.exports = {
    format: format
}
```