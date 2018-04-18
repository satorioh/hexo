---
title: 【ES6】let、const、解构、字符串、数值的扩展
date: 2018-04-18 09:13:20
categories: ES6
tags:
- js
- 前端
permalink: es6-let-string-number-destruction
---
### 一、let和const命令
#### 1.特点：
- 块级作用域，防止污染全局变量
- 没有变量提升，防止意外错误
- 不可重复声明变量
- 不可在声明前使用该变量
<!--more-->
#### 2.注意：
- const本质是保证变量指向的内存地址不得改变，所以对于对象，对象的属性依旧可以更改
- let、const声明的变量，不再默认是全局对象的属性

#### 3.示例：
- for循环中的i用let声明
- 经典闭包问题解决

### 二、变量的解构赋值
#### 1.语法：
##### （1）数组的解构赋值
```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

##### （2）默认值：
```javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'

var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```

##### (3)对象的解构赋值
```javascript
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined

let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };

let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```
##### (4)字符串的解构赋值
```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```
##### (5)函数参数的解构赋值
```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

#### 2.注意：
- 只有变量对应的值显示设置为undefined，默认值才会生效，如果为null，则不生效
- 对象的解构赋值，变量必须与属性同名，才能取到正确的值

#### 3.示例：
##### (1)数组解构
```javascript
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

##### (2)交换变量的值
```javascript
let x = 1;
let y = 2;
[x, y] = [y, x];
```
##### (3)从函数返回多个值
```javascript
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```
##### (4)提取JSON数据
```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

### 三、字符串的扩展
#### 1.新增方法：
##### (1)includes:返回布尔值，表示是否找到了参数字符串
##### (2)startsWith:返回布尔值，表示参数字符串是否在原字符串的头部
##### (3)endsWith:返回布尔值，表示参数字符串是否在原字符串的尾部
```javascript
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```
(4)repeat:返回一个新字符串，表示将原字符串重复n次
```javascript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```
(5)padStart:头部补全
(6)padEnd:尾部补全

```javascript
//为数值不全指定位数
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"

//提示字符串格式
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

#### 2.注意
- includes、startsWith、endWith这三个方法都支持第二个参数，表示开始搜索的位置
- 使用第二个参数n时，endsWith的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束

### 四、数值的扩展
#### 1.新增方法：
##### (1)ES6 提供了二进制和八进制数值的新的写法，分别用前缀0b（或0B）和0o（或0O）表示
##### (2)Number.isFinite:用来检查一个数值是否为有限的
##### (3)Number.isNaN:用来检查一个值是否为NaN
```javascript
isFinite(25) // true
isFinite("25") // true
Number.isFinite(25) // true
Number.isFinite("25") // false

isNaN(NaN) // true
isNaN("NaN") // true
Number.isNaN(NaN) // true
Number.isNaN("NaN") // false
Number.isNaN(1) // false
```
##### (4)Number.parseInt:与原方法同
##### (5)Number.parseFloat:与原方法同
##### (6)Number.isInteger:判断一个数值是否为整数
##### (7)Math.trunc:用于去除一个数的小数部分，返回整数部分
##### (8)Math.sign:用来判断一个数到底是正数、负数、还是零。对于非数值，会先将其转换为数值。
##### (9)新增了一个指数运算符（**）
```javascript
let a = 1.5;
a **= 2;
// 等同于 a = a * a;

let b = 4;
b **= 3;
// 等同于 b = b * b * b;
```

#### 2.注意：
- Number.isFinite、Number.isNaN与传统方法区别：传统方法先调用Number()将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，Number.isFinite()对于非数值一律返回false, Number.isNaN()只有对于NaN才返回true，非NaN一律返回false
