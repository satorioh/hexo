---
title: 【JS】match、search、replace、split、exec、test对比总结
date: 2017-12-26 11:16:37
categories: JavaScript
tags:
- js
- 前端
permalink: js-match-search-replace-split-exec-test
---
### 一、String中支持正则的方法：
#### **1.match**:返回包含一个/所有匹配结果的数组，没有则返回null

**语法：**
```javascript
str.match(regexp)
```
**非全局模式：**返回和exec()相同结果。一个数组，其中只有第一个匹配项，额外的属性index表示匹配结果在原字符串中的索引，input属性表示被解析的原始字符串<!--more-->
```javascript
var a = 'aaaa'.match(/\w/);
console.log(a); // ["a", index: 0, input: "aaaa"]
```
**全局模式：**返回一个数组，包含所有的匹配项，没有index或input属性
```javascript
var a = 'aaaa'.match(/\w/g);
console.log(a); // ["a", "a", "a", "a"]
```

#### **2.search**:返回首个匹配项的索引，没有则返回-1

**语法：**
```javascript
str.search(regexp)
```
**注意：**search方法不执行全局匹配，它将忽略标志g。它同时忽略 regexp的lastIndex属性，并且总是从字符串的开始进行检索

#### **3.replace**:用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。

**语法：**
```javascript
str.replace(regexp|substr, newSubStr|function)
```
**注意：**replace方法并不改变调用它的字符串本身，而只是返回一个新的替换后的字符串

**字符串/非全局模式：**只替换第一个匹配项
```javascript
'aaaa'.replace('a', 'b')     //"baaa"
'aaaa'.replace(/\w/, 'b')    //"baaa"
```
**全局模式：**替换所有匹配项
```javascript
'aaaa'.replace(/\w/g, 'b')    //"bbbb"
```
**特殊变量名举例：**交换两个单词的位置
```javascript
var re = /(\w+)\s(\w+)/;
var str = "John Smith";
var newstr = str.replace(re, "$2, $1");
// Smith, John
console.log(newstr);
```
**第二个参数为函数时：**函数的返回值将作为替换字符串；该回调函数的参数：

| 变量名    | 代表的值                                                                                          |
|:---------|:-------------------------------------------------------------------------------------------------|
| match    | 符合正则匹配的子串                                                                                  |
| p1,p2... | 捕获组，对应于特殊变量的$1，$2等                                                                     |
| offset   | 匹配到的子字符串在原字符串中的偏移量。（比如，如果原字符串是“abcd”，匹配到的子字符串是“bc”，那么这个参数将是1） |
| string|原字符串|

```javascript
'aaaa'.replace(/\w/g, function() {
    return 'b';
}); // "bbbb"

'aaaa'.replace(/\w/g, function(match) {
    return match.toUpperCase();
}); // "AAAA"
```

#### **4.split**:使用指定的分隔符字符串将一个String对象分割成字符串数组
**语法：**
```javascript
str.split([separator[, limit]])
```
**举例：**限制返回值中分割元素数量
```javascript
var myString = "Hello World. How are you doing?";
var splits = myString.split(" ", 3);
console.log(splits); //["Hello", "World.", "How"]
```
### 二、RegExp中的API：
#### **5.exec:**在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 null
**语法：**
```javascript
regexp.exec(str)
```
第一个元素是与整个正则匹配的子字符串；第二个元素是捕获组...index和input同match方法

**举例：**
```javascript
var result = /(\d+)-(\w+)/.exec('12-ab');
console.log(result) // --> ["12-ab", "12", "ab", index: 0, input: "12-ab"]
```

**非全局模式：**每次实例的lastIndex属性的值总是不变的
```javascript
var reg = /\d/;
//第一次匹配
console.log(reg.exec('a123'));
console.log(reg.lastIndex);
//输出
["1"]
  0
  
第二次匹配
console.log(reg.exec('a123'));
console.log(reg.lastIndex);
//输出
["1"]
  0 
```
**全局模式：**每次实例的lastIndex属性的值为匹配文本最后一个字符的下一个位置，当 exec() 再也找不到匹配的文本时，它将返回 null，并把 lastIndex 属性重置为 0
```javascript
var reg = /\d/g;
//第一次匹配
console.log(reg.exec('a123'));
console.log(reg.lastIndex);
//输出
["1"]
  2
  
第二次匹配
console.log(reg.exec('a123'));
console.log(reg.lastIndex);
//输出
["2"]
  3 

第三次匹配
console.log(reg.exec('a123'));
console.log(reg.lastIndex);
//输出
["3"]
  4 

第四匹配
console.log(reg.exec('a123'));
console.log(reg.lastIndex);
//输出
null
  0
```
**使用循环，获取全部匹配项：**
```javascript
var reg = /\d/g,
    result = [],
    crt;
while((crt = reg.exec('a123')) !== null){
    result = result.concat(crt)
};
result; //["1", "2", "3"]
```

#### **6.test:**执行一个检索，用来查看正则表达式与指定的字符串是否匹配。返回 true 或 false
**语法：**
```javascript
regexp.test(str)
```
**非全局模式：**lastIndex不变
```javascript
/\d/.test('asdf2') // --true   检测字符串`'asdf2'`中是否包含数字
```
**全局模式：**连续的执行test()方法，后续的执行将会从 lastIndex 处开始匹配字符串
```javascript
var regex = /foo/g;

// regex.lastIndex is at 0
regex.test('foo'); // true

// regex.lastIndex is now at 3
regex.test('foo'); // false
```

参考：

[[ JS 进阶 ] test, exec, match, replace](https://segmentfault.com/a/1190000003497780)

[MDN](https://developer.mozilla.org/zh-CN/)

