---
title: 【JS】常用继承方式总结
date: 2017-12-07 17:06:50
categories: JavaScript
tags:
- js
- 前端
permalink: inheritance-method-summary
---
### 方式一：原型链继承（prototype模式）
```javascript
function Animal(){
    this.species = "动物";
}
function Cat(name,color){
    this.name = name;
    this.color = color;
}

Cat.prototype = new Animal();//核心
Cat.prototype.constructor = Cat;
```
<!--more-->
缺点：

1.原型上的引用类型的属性和方法被所有实例共享（个人觉得这不应该算缺点，毕竟原型就是派这个用处的）

2.创建Cat的实例时，无法向Animal传参

### 方式二：经典继承（构造函数绑定）
```javascript
function Animal(){
    this.species = "动物";
}
function Cat(name,color){
    Animal.call(this);//核心，或Animal.apply(this,arguments);
    this.name = name;
    this.color = color;
}
```
优点：

1.避免了引用类型的属性和方法被所有实例共享，因为根本没用到原型嘛

2.创建Cat的实例时，可以向Animal传参

缺点：

1.属性和方法都在构造函数中定义，每次创建实例都会创建一遍属性和方法

### 方式三：组合继承（原型链继承和经典继承双剑合璧）
```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {

    Parent.call(this, name);//核心
    
    this.age = age;

}

Child.prototype = new Parent();//核心

var child1 = new Child('kevin', '18');

child1.colors.push('black');

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

var child2 = new Child('daisy', '20');

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```
优点：

1.融合原型链继承和构造函数的优点融合原型链继承和构造函数的优点

缺点：

1.两次调用Parent构造函数，在Child.prototype上增加了额外、不需要的属性

### 方式四：寄生组合继承
```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);//核心
    this.age = age;
}

Child.prototype = Object.create(Parent.prototype);//核心
Child.prototype.constructor = Child;

var child1 = new Child('kevin', '18');

console.log(child1)
```
优点：
在组合继承的基础上，避免了在 Child.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf