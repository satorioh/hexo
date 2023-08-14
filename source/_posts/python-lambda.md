---
title: lambda函数
date: 2023-01-31 13:50:30
categories: Python
tags:
- python
permalink: python-lambda
---
### 一、语法
```python
lambda argument_list:expersion
```
注意1：lambda函数体只能写一句话

注意2：不支持赋值语句，如下代码报错
```python
func01 = lambda p: p = 10
```
<!--more-->

### 二、使用

#### 1.定义为函数
##### (1)有参数有返回值

```python
add = lambda x,y: x+y
add(1,4) # 5
```
##### (2)无参数有返回值
```python
func01 = lambda: True
print(func01())
```
##### (3)有参数无返回值
```python
func01 = lambda p: print(p)
func01(123)
```
##### (4)无参数无返回值
```python
func01 = lambda: print("hello")
func01()
```

#### 2.即时定义即时使用

```python
(lambda x,y: x+y)(3,5) # 8
```

#### 3.结合map、filter、reduce、sorted

```python
# 作为 map 的迭代方法
a = [1, 2, 3, 4, 5, 6]
result = map(lambda x: x+1, a)
list(result)
# [2, 3, 4, 5, 6, 7]

# 指定属性排序
s = [{'name': 'tom', 'age': 22},
     {'name': 'lily', 'age': 19},
     {'name': 'lucy', 'age': 20}]

sorted(s, key=lambda x: x['age'])

# [{'name': 'lily', 'age': 19},
#  {'name': 'lucy', 'age': 20},
#  {'name': 'tom', 'age': 22}]
```

#### 4.条件判断

```python
# 两个数的最大值
(lambda x,y: x if x>y else y )(49,5) # 49
```

#### 5.和字典结合

```python
# 可以定义在字典的值里，用 key 来调用
d = {'+': lambda x,y: x+y, '-': lambda x,y: x-y}
d['+'](3, 8) # 11
```
