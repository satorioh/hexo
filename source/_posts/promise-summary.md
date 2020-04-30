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
与all()唯一的不同在于, 它不会进行短路, 也就是说当Promise全部处理完成后,我们可以拿到每个Promise的状态, 而不管是否处理成功

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

##### 6.Promise.resolve(value)
类方法，该方法返回一个以value值解析后的Promise对象

**返回：**

1.如果传入的value本身就是Promise对象，则该对象作为Promise.resolve方法的返回值返回。
```javascript
function fn(resolve){
    setTimeout(function(){
        resolve(123);
    },3000);
}
let p0 = new Promise(fn);
let p1 = Promise.resolve(p0);
console.log(p0 === p1);// 返回为true，返回的Promise即是入参的Promise对象
```

2.如果这个值是个thenable(即带有 then 方法)，返回的Promise对象会“跟随”这个 thenable的对象，采用它的最终状态（指 resolved/rejected/pending/settled）
```javascript
//如果传入的 value 本身就是 thenable 对象，返回的 promise 对象会跟随 thenable 对象的状态。
let promise = Promise.resolve($.ajax('/test/test.json'));// => promise对象
promise.then(function(value){
   console.log(value);
});
```

3.其他情况以该值为成功状态返回一个 Promise对象。
```javascript
let p1 = Promise.resolve(123);
//打印p1 可以看到p1是一个状态置为resolved的Promise对象
console.log(p1)
```

#### 二、关于return
1.链式中，后者的状态取决于前者（成功/失败）的回调函数中返回（return）的结果。

- 如果没有返回，相当返回一个成功的状态，值为undefined。

- 如果返回为Promise对象，后者的状态由该对象的最终状态决定。

- 如果返回为非Promise对象的数据，相当返回一个成功的状态，值为此数据。

- 如果前者执行时抛出了错误，相当是返回一个失败的状态，值为此错误。

2.在链式中，如果前者的状态没有被后者捕获，会一直（像）冒泡到被捕获为止。
```javascript
new Promise(resolve => resolve(2000))
  .then(null, err => console.log('err', err))
  .then(res => console.log('res', res)) // res 2000
```

3.then 或者 .catch 的参数期望是函数，传入非函数则会发生值穿透
```javascript
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log) // 1
```

#### 三、应用
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
[面试精选之Promise](https://juejin.im/post/5b31a4b7f265da595725f322)
