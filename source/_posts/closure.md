---
title: 【JS】闭包
date: 2018-04-11 11:44:01
categories: JavaScript
tags:
- js
- 前端
permalink: closure
---
以下内容为我的理解，仅供参考

#### 一、什么是闭包
有权访问其他函数内部自由变量的**函数**，就是闭包
<!--more-->
#### 二、闭包的原理与实现
定义一个外部函数outer，在其内部有一个局部变量a，一个内层函数inner，inner在其作用域链上可以访问并操作a，外部函数outer将inner作为返回值返回，此时将此返回值（inner）赋值给一个全局变量b，因为b永远保持在内存中，所以对应的inner-->a-->outer这一变量环境也被一并保存住，即形成闭包。

#### 三、闭包优缺点
优点：保有私有变量，方便访问，防止全局污染；变量重用，防止被非法篡改

缺点：占用更多内存空间

#### 四、示例：闭包计数器与私有方法
```javascript
function createCounter(counterName) 
{
  var counter = 0;

  function display() 
  {
    console.log("Number of " + counterName + ": " + counter);
  }

  function increment() 
  {
    counter = counter + 1;
    display();
  };

  function decrement() 
  {
    counter = counter - 1;
    display();
  };

  return {
    increment : increment,
    decrement : decrement
  };
}

var dogsCounter = createCounter("dogs");

dogsCounter.increment(); // Number of dogs: 1
dogsCounter.increment(); // Number of dogs: 2
dogsCounter.decrement(); // Number of dogs: 1
```

参考链接：

[解密JavaScript闭包](https://blog.fundebug.com/2017/07/31/javascript-closure/)