---
title: 【CSS】布局方案总结
date: 2018-03-28 15:47:21
categories: CSS
tags:
- css
- 前端
permalink: css-layout-summary
---
## 一、居中布局
### 1.水平居中
#### （1）text-align:center
- 父元素：块
- 子元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.child{
    display:inline/inline-block;
}
.parent{
    text-align:center;
}
```
- 优点：兼容性好，甚至可以兼容ie6、ie7
- 缺点：child里的文字也会水平居中，可以在.child添加text-align:left;还原

#### （2）margin:0 auto;
- 父元素：块
- 子元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.child {
    width:100px;
    margin:0 auto;
}
```

#### （3）absolute + transform
- 父元素：块
- 子元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    position:relative;
}
.child {
    position:absolute;
    left:50%;
    transform:translateX(-50%);
}
```
- 优点：居中元素不会对其他的产生影响
- 缺点：transform属于css3内容，兼容性存在一定问题，高版本浏览器需要添加一些前缀

#### （4）flex + justify-content
- 父元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    display:flex;
    justify-content:center;
}
```
- 优点：设置parent即可
- 缺点：低版本浏览器(ie6 ie7 ie8)不支持

#### （5）浮动元素水平居中
- 父元素：块
- 子元素：块
```html
<div class="box">
    <p>我是浮动的</p>
    <p>我也是居中的</p>
</div>

.box{
    float:left;
    position:relative;
    left:50%;
}
p{
    float:left;
    position:relative;
    right:50%; //或left: -50%
}
```

### 2.垂直居中
#### （1）table-cell + vertical-align
- 父元素：块
- 子元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    display:table-cell;
    vertical-align:middle;
}
```
- 优点：兼容性较好，ie8以上均支持

#### （2）absolute + transform
- 父元素：块
- 子元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    position:relative;
}
.child {
    position:absolute;
    top:50%;
    transform:translateY(-50%);
}
```
- 优点：居中元素不会对其他的产生影响
- 缺点：transform属于css3内容，兼容性存在一定问题，高版本浏览器需要添加一些前缀

#### （3）flex + align-items
- 父元素：块
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    position:flex;
    align-items:center;
}
```
- 优点：只设置parent
- 缺点：兼容性问题

#### （4）line-height法
- 父元素：块，定高
- 子元素：单行的button、图片或者文本
```html
#father{
    height:100px;
    line-height:100px;
    background:#999;
}
```

#### （4）多行文本垂直居中
```html
.box {line-height:300px;}
.box>span {
    display:inline-block;
    line-height:normal;
    vertical-align:middle;
}
```

### 3.水平垂直居中
#### （1）absolute + transform
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    position:relative;
}
.child {
    position:absolute;
    left:50%;
    top:50%;
    transform:tranplate(-50%,-50%);
}
```

#### （2）inline-block + text-align + table-cell + vertical-align
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    text-align:center;
    display:table-cell;
    vertical-align:middle;
}
.child {
    display:inline-block;
}
```

#### （3）flex + justify-content + align-items
```html
<div class="parent">
    <div class="child>DEMO</div>
</div>

.parent {
    display:flex;
    justify-content:center;
    align-items:center;
}
```

## 二、多列布局