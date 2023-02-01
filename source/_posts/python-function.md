---
title: 函数
date: 2023-02-01 13:45:15
categories: Python
tags:
- python
permalink: python-function
---
#### 1.参数的默认值

```python
def search4letters(phrase: str, letters: str = 'aeiou') -> set:
    """在指定字符串中找到输入的字母"""
    return set(letters) & set(phrase)
```

<!--more-->

#### 2.关键字参数: 关键字参数后面必须都是关键字参数

```python
def is_triangle(a, b, c):
    print(f'a = {a}, b = {b}, c = {c}')
    return a + b > c and b + c > a and a + c > b

# 调用函数传入参数不指定参数名按位置对号入座
print(is_triangle(1, 2, 3))
# 调用函数通过“参数名=参数值”的形式按顺序传入参数
print(is_triangle(a=1, b=2, c=3))
# 调用函数通过“参数名=参数值”的形式不按顺序传入参数
print(is_triangle(c=3, a=1, b=2))
```

#### 3.可变参数: 使用*args或**kwargs传递

```python
# 用星号表达式来表示args可以接收0个或任意多个参数
def foo(*t):
    print(t)
# 调用
foo(1) # (1,)
foo(1,4,5,6) # (1, 4, 5, 6)
```

```python
def say(name, age=18, words='hello'):
    print(f'{age}岁的{name}说：{words}')
# 调用
d = {'name':'tom', 'age':20, 'words': 'hello'}
say(**d) # 20岁的tom说：hello
```

#### 4.命名关键字参数：参数列表中的*是一个分隔符，*前面的参数都是位置参数，而*后面的参数就是命名关键字参数

```python
def is_triangle(*, a, b, c):
    print(f'a = {a}, b = {b}, c = {c}')
    return a + b > c and b + c > a and a + c > b

# TypeError: is_triangle() takes 0 positional arguments but 3 were given
# print(is_triangle(3, 4, 5))
# 传参时必须使用“参数名=参数值”的方式，位置不重要
print(is_triangle(a=3, b=4, c=5))
print(is_triangle(c=5, b=4, a=3))
```
