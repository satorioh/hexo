---
title: Promise总结
date: 2020-04-28 13:42:57
categories: JavaScript
tags:
- js
permalink: promise-summary
---
#### 一、方法
##### 1.finally()
finally()方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。它总是会返回原来的值
```javascript
const p = Promise.resolve(2).finally(() => {});
p.then(value => console.log(value)) // 2

const p = Promise.reject(3).finally(() => {},() => {});
p.catch(reason => console.log(reason)) // 3
```
<!--more-->

##### 2.all()
**语法：**
```javascript
const p = Promise.all([p1, p2, p3]);
```

**状态：**
>fulfilled：p1, p2, p3都为fulfilled
>
>rejected: p1, p2, p3其中有一个为rejected

**返回值：**
>fulfilled：由p1、p2、p3的返回值组成的一个数组
>
>rejected: 第一个被reject的实例的返回值

##### 3.race()
**语法：**
```javascript
const p = Promise.race([p1, p2, p3]);
```

**状态：**
>fulfilled：p1, p2, p3中第一个变为fulfilled
>
>rejected: p1, p2, p3中第一个变为rejected

**返回值：**
>fulfilled：p1, p2, p3中第一个变为fulfilled的返回值
>
>rejected: p1, p2, p3中第一个变为rejected的返回值

**示例：指定时间内没有获得结果，就报错**
```javascript
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);

p
.then(console.log)
.catch(console.error);
```

##### 4.allSettled()
不关心异步操作的结果，只关心这些操作有没有结束

**语法：**
```javascript
const p = Promise.allSettled([p1, p2, p3]);
```

**状态：**
>fulfilled：p1, p2, p3全部完成(fulfilled/rejected)
>
>rejected: 不存在此状态

**返回值：**
>fulfilled：Promise实例组成的数组

**示例：**
```javascript
const resolved = Promise.resolve(42);
const rejected = Promise.reject(-1);

const allSettledPromise = Promise.allSettled([resolved, rejected]);

allSettledPromise.then(function (results) {
  console.log(results);
});
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```

##### 5.any()
该方法目前是一个第三阶段的提案

**语法：**
```javascript
const p = Promise.any([p1, p2, p3]);
```

**状态：**
>fulfilled：p1, p2, p3中第一个变为fulfilled
>
>rejected: p1, p2, p3全部变为rejected

**返回值：**
>fulfilled：p1, p2, p3中第一个变为fulfilled的返回值
>
>rejected: 由错误信息组成的数组

**示例：**
```javascript
var resolved = Promise.resolve(42);
var rejected = Promise.reject(-1);
var alsoRejected = Promise.reject(Infinity);

Promise.any([resolved, rejected, alsoRejected]).then(function (result) {
  console.log(result); // 42
});

Promise.any([rejected, alsoRejected]).catch(function (results) {
  console.log(results); // [-1, Infinity]
});
```

#### 二、应用
##### 1.异步加载图片
```javascript
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
```

##### 2.包装ajax操作
```javascript
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};
```

参考文章：

[ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/promise)
