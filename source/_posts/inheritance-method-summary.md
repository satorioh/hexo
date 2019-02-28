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
function Parent (name) {
  this.name = name
  this.colors = ['red','green']
}

Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child (age) {
  this.age = age
}

Child.prototype = new Parent()
Child.prototype.constructor = Child

var child1 = new Child ('20')
console.log(child1.getName) // ƒ () {console.log(this.name)}
var child2 = new Child ('20')
child2.colors.push('blue')
console.log(child1.colors) //['red','green','blue']
console.log(child2.colors) //['red','green','blue']
```
<!--more-->
缺点：

1.引用类型的属性被所有实例共享

2.创建Child的实例时，无法向Parent传参

### 方式二：经典继承（构造函数绑定）
```javascript
function Parent (name) {
  this.name = name
  this.colors = ['red','green']
}

Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child (age, name) {
  Parent.call(this, name) //核心
  this.age = age
}

var child1 = new Child ('20','bill')
console.log(child1.getName) //undefined
var child2 = new Child ('20','lily')
child2.colors.push('blue')
console.log(child1.colors) //['red','green']
console.log(child2.colors) //['red','green','blue']
```
优点：

1.避免了引用类型的属性被所有实例共享（因为根本没用到原型链）

2.创建Child的实例时，可以向Parent传参

缺点：

1.属性都在构造函数中定义，每次创建实例都会创建一遍属性

2.没有继承到Parent原型链上的方法

### 方式三：组合继承（原型链继承和经典继承双剑合璧）
```javascript
function Parent (name) {
  this.name = name
  this.colors = ['red','green']
}

Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child (age, name) {
  Parent.call(this, name) //核心
  this.age = age
}

Child.prototype = new Parent()
Child.prototype.constructor = Child

var child1 = new Child ('20','bill')
console.log(child1.getName) //undefined
var child2 = new Child ('20','lily')
child2.colors.push('blue')
console.log(child1.colors) //['red','green']
console.log(child2.colors) //['red','green','blue']
console.log(Child.prototype) //{name: undefined, colors: Array(2), constructor: ƒ}
```
优点：

1.融合原型链继承和构造函数的优点融合原型链继承和构造函数的优点

缺点：

1.两次调用Parent构造函数，在Child.prototype上增加了额外、不需要的属性

### 方式四：寄生组合继承
```javascript
function Parent (name) {
  this.name = name
  this.colors = ['red','green']
}

Parent.prototype.getName = function () {
  console.log(this.name)
}

function Child (age, name) {
  Parent.call(this, name) //核心
  this.age = age
}

Child.prototype = Object.create(Parent.prototype) //核心
Child.prototype.constructor = Child

var child1 = new Child ('20','bill')
console.log(child1.getName) //undefined
var child2 = new Child ('20','lily')
child2.colors.push('blue')
console.log(child1.colors) //['red','green']
console.log(child2.colors) //['red','green','blue']
console.log(Child.prototype) //{constructor: ƒ}
```
优点：
在组合继承的基础上，避免了在 Child.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf