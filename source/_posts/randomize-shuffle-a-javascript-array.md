---
title: 【JS】数组取随机数
date: 2018-01-10 11:20:33
categories: JavaScript
tags:
- js
- 前端
permalink: randomize-shuffle-a-javascript-array
---
## 一、使用数组sort方法对数组元素随机排序，并取指定长度的元素
```javascript
Array.prototype.shuffle = function (n) {
  var len = this.length,
      num = n ? Math.min(n, len) : len,
      arr = this.slice(0);
  arr.sort(function (a, b) { return Math.random() - 0.5; });
  return arr.slice(0, num - 1);
};
```
<!--more-->

## 二、随机交换数组内的元素
```javascript
lib = {};
lib.range = function (min, max) { return min + Math.floor(Math.random() * (max - min + 1)); };

Array.prototype.shuffle = function (n) {
  var len = this.length,
      num = n ? Math.min(n, len) : len,
      arr = this.slice(0), 
      temp, 
      index;
  for (var i = 0; i < len; i++) {
    index = lib.range(i, len - 1);
    temp = arr[i];
    arr[i] = arr[index];
    arr[index] = temp;
  }
  return arr.slice(0, num);
};
```

## 三、随机从原数组抽取一个元素,加入到新数组
```javascript
lib = {};
lib.range = function (min, max) { return min + Math.floor(Math.random() * (max - min + 1)); };
Array.prototype.shuffle = function (n) {
  var len = this.length,
    num = n ? Math.min(n, len) : len,
    arr = this.slice(0),
    result = [],
    index;
  for (var i = 0; i < num; i++) {
    index = lib.range(0, len - 1 - i);
    // 或者 result.concat(arr.splice(index,1)) 
    result.push(arr.splice(index, 1)[0]);
  }
  return result;
};
```

## 简易版本
```javascript
"use strict";
let arr = [1, 2, 3, 4, 5, 6];
console.log(arr.sort(() => Math.random() - 0.5)[0]);
```

参考：[JavaScript学习笔记：数组随机排序](https://www.w3cplus.com/javascript/how-to-randomize-shuffle-a-javascript-array.html "JavaScript学习笔记：数组随机排序")
