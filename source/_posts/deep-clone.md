---
title: 【JS】深拷贝
date: 2019-01-06 15:35:55
categories: JavaScript
tags:
- js
- 前端
permalink: deep-clone
---
### 一、赋值、浅拷贝与深拷贝的区别
![区别](../images/clone.png)
<!--more-->

### 二、深拷贝的方法
#### 1.JSON转换
```javascript
var targetObj = JSON.parse(JSON.stringify(copyObj))
let arr4 = JSON.parse(JSON.stringify(arr))
```
缺点：

（1）如果对象里有函数,函数无法被拷贝下来

（2）无法拷贝copyObj对象原型链上的属性和方法

（3）当数据的层次很深，会栈溢出

#### 2.普通递归函数
```javascript
function deepCopy( source ) {
if (!isObject(source)) return source; //如果不是对象的话直接返回
    let target = Array.isArray( source ) ? [] : {} //数组兼容
    for ( var k in source ) {
    	if (source.hasOwnProperty(k)) {
    		if ( typeof source[ k ] === 'object' ) {
            	target[ k ] = deepCopy( source[ k ] )
        	} else {
            	target[ k ] = source[ k ]
        	}
    	}
    }
    return target
}

function isObject(obj) {
    return typeof obj === 'object' && obj !== null
}
```

缺点：

（1）无法保持引用

（2）当数据的层次很深，会栈溢出

#### 3.防栈溢出函数
```javascript
function cloneLoop(x) {
    const root = {};

    // 栈
    const loopList = [
        {
            parent: root,
            key: undefined,
            data: x,
        }
    ];

    while(loopList.length) {
        // 深度优先
        const node = loopList.pop();
        const parent = node.parent;
        const key = node.key;
        const data = node.data;

        // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }

        for(let k in data) {
            if (data.hasOwnProperty(k)) {
                if (typeof data[k] === 'object') {
                    // 下一次循环
                    loopList.push({
                        parent: res,
                        key: k,
                        data: data[k],
                    });
                } else {
                    res[k] = data[k];
                }
            }
        }
    }

    return root;
}
```

优点：

（1）不会栈溢出

（2）支持很多层级的数据

参考文章：

[深拷贝的终极探索](https://yanhaijing.com/javascript/2018/10/10/clone-deep/)

[一篇文章彻底说清JS的深拷贝/浅拷贝](https://segmentfault.com/a/1190000012828382)

[浅拷贝与深拷贝](https://github.com/ljianshu/blog/issues/5)

[面试题：如何实现一个深拷贝](https://juejin.im/post/5c45112e6fb9a04a027aa8fe)
