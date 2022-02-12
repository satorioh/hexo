---
title: 【Vue】修饰符
date: 2018-07-03 09:02:56
categories: VUE
tags:
- vue
- 前端
permalink: vue-modifiers
---
#### 一、事件修饰符
1.`.stop`: 等同于event.stopPropagation()
```html
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>
```
2.`.prevent`:等同于event.preventDefault()
```html
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>
```
<!--more-->
3.`.capture`:使用事件捕获模式
```html
<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>
```
4.`.self`:只当在 event.target 是当前元素自身时触发处理函数
```html
<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>
```
5.`.once`:点击事件将只会触发一次。支持自定义事件
```html
<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>
```
6.`.passive`:事件的默认行为不会被取消，提升移动端性能
```html
<!-- the scroll event will not cancel the default scroll behavior -->
<div v-on:scroll.passive="onScroll">...</div>
```

#### 二、按键修饰符：监测相应键盘事件时，触发如下按键，才会调用函数
1`.enter`

2.`.tab`

3.`.delete`:删除和退格键

4.`.esc`

5.`.space`

6.`.up`

7.`.down`

8.`.left`

9.`.right`

10.`.left`:鼠标左键

11.`.right`:鼠标右键

12.`.middle`:鼠标中键

#### 三、系统修饰符：需要按住相应按键才触发鼠标或键盘事件的监听器
1.`.ctrl`

2.`.alt`

3.`.shift`

4.`.meta`：Windows下是徽标键，Mac下是commond键

5.`.exact`: 有且只有特定的按键被触发时，才调用函数
```html
<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>
```

#### 四、表单v-model修饰符
1.`.lazy`:v-model绑定后，默认input是同步输入框数据的，使用.lazy后，只在失去焦点，或按回车后才更新
```html
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```
2.`.number`:自动将用户的字符串输入值转为数值类型
```html
<input v-model.number="age" type="number">
```
3.`.trim`:自动过滤用户输入的首尾空白字符
