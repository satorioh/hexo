---
title: 【CSS】父子元素margin-top重叠问题解决
date: 2017-05-06 16:49:15
categories: CSS
tags:
- css
- 前端
permalink: margin-top-overlap-problem
---
**问题代码：**
```html
<div id="father" style="background:#eee;">
   <div id="son" style="margin-top:50px;">son</div>
</div>
```
<!--more-->
**方法一：** 父元素设置overflow:hidden
```html
<div id="father" style="background:#eee;overflow:hidden;">
   <div id="son" style="margin-top:50px;">son</div>
</div>
```

**方法二：** 父元素设置border-top
```html
<div id="father" style="background:#eee;border-top:1px solid transparent;">
   <div id="son" style="margin-top:50px;">son</div>
</div>
```

**方法三：** 父元素设置padding-top
```html
<div id="father" style="background:#eee;padding-top:1px;">
   <div id="son" style="margin-top:50px;">son</div>
</div>
```

**方法四：** 父子元素间增加一个inline元素
```html
<div id="father" style="background:#eee;">&nbsp;
   <div id="son" style="margin-top:50px;">son</div>
</div>
```

**方法五：** 内容生成
```css
.father:before {
    content:' ';
    display:table;
}
```
