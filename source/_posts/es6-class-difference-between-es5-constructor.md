---
title: 【JS】ES6 class和普通构造函数的区别
date: 2018-06-04 11:16:24
categories: JavaScript
tags:
- js
- 前端
permalink: es6-class-difference-between-es5-constructor
---
1.类的内部所有定义的方法，都是不可枚举的（但是在es5中prototype的方法是可以进行枚举的）

2.类的构造函数，不使用new是没法调用的，会报错。这是它跟普通构造函数的一个主要区别，后者不用new也可以执行

3.Class不存在变量提升（hoist），这一点与ES5完全不同
<!--more-->

4.ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。

```javascript
// ES6
class Parent{
    constructor(name){
        this.name = name;
    }
    static sayHello(){
        console.log('hello');
    }
    sayName(){
        console.log('my name is ' + this.name);
        return this.name;
    }
}
class Child extends Parent{
    constructor(name, age){
        super(name);
        this.age = age;
    }
    sayAge(){
        console.log('my age is ' + this.age);
        return this.age;
    }
}
let parent = new Parent('Parent');
let child = new Child('Child', 18);
console.log('parent: ', parent); // parent:  Parent {name: "Parent"}
Parent.sayHello(); // hello
parent.sayName(); // my name is Parent
console.log('child: ', child); // child:  Child {name: "Child", age: 18}
Child.sayHello(); // hello
child.sayName(); // my name is Child
child.sayAge(); // my age is 18
```

参考链接：

[ES6 class创建对象和传统方法生成对象的区别](http://www.fly63.com/article/detial/417)

[Class 的基本语法](http://es6.ruanyifeng.com/#docs/class)