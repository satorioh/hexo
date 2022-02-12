---
title: 【CSS】冒泡loading样式
date: 2019-04-11 09:59:19
categories: CSS
tags:
- css
- 前端
permalink: css-spinner-bounce
---
#### 样式一：水平三个冒泡
```html
<div class="spinner">
  <div class="bounce1"></div>
  <div class="bounce2"></div>
  <div class="bounce3"></div>
</div>
```
<!--more-->
```css
.spinner > div {
    width: 30px;
    height: 30px;
    background-color: #3a84d8;

    border-radius: 100%;
    display: inline-block;
    -webkit-animation: bouncedelay 1.4s infinite ease-in-out;
    animation: bouncedelay 1.4s infinite ease-in-out;
    /* Prevent first frame from flickering when animation starts */
    -webkit-animation-fill-mode: both;
    animation-fill-mode: both;
  }

  .spinner .bounce1 {
    -webkit-animation-delay: -0.32s;
    animation-delay: -0.32s;
  }

  .spinner .bounce2 {
    -webkit-animation-delay: -0.16s;
    animation-delay: -0.16s;
  }

  @-webkit-keyframes bouncedelay {
    0%, 80%, 100% { -webkit-transform: scale(0.0) }
    40% { -webkit-transform: scale(1.0) }
  }

  @keyframes bouncedelay {
    0%, 80%, 100% {
      transform: scale(0.0);
      -webkit-transform: scale(0.0);
    } 40% {
        transform: scale(1.0);
        -webkit-transform: scale(1.0);
      }
  }
```

#### 样式二：同心圆两个冒泡
```html
<div class="spinner">
  <div class="double-bounce1"></div>
  <div class="double-bounce2"></div>
</div>
```
```css
.spinner {
  width: 20px;
  height: 20px;
  position: relative;
}

.double-bounce1,
.double-bounce2 {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background-color: #3063b2;
  opacity: 0.6;
  position: absolute;
  top: 0;
  left: 0;
  -webkit-animation: bounce 2.0s infinite ease-in-out;
  animation: bounce 2.0s infinite ease-in-out;
}

.double-bounce2 {
  -webkit-animation-delay: -1.0s;
  animation-delay: -1.0s;
}

@-webkit-keyframes bounce {

  0%,
  100% {
    -webkit-transform: scale(0.0)
  }

  50% {
    -webkit-transform: scale(1.0)
  }
}

@keyframes bounce {

  0%,
  100% {
    transform: scale(0.0);
    -webkit-transform: scale(0.0);
  }

  50% {
    transform: scale(1.0);
    -webkit-transform: scale(1.0);
  }
}
```
