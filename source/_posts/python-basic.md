---
title: Python基础常用技巧
date: 2022-10-10 11:18:16
categories: Python
tags:
- python
permalink: python-basic
---
#### 1.数据类型

整型int、浮点数float、字符串str、布尔bool(True、False)

#### 2.print格式化输出

```python
name = input('please enter your name: ')
print('hello,', name)

print('%.1f华氏度 = %.1f摄氏度' % (f, c))
# 或者
print(f'{f:.1f}华氏度 = {c:.1f}摄氏度')
```

<!--more-->

#### 3.循环

```python
# 用for循环实现1~100之间的偶数求和
total = 0
for x in range(2, 101, 2):
    total += x
print(total)
```

#### 4.三元运算

```python
X = 10 if y > 3 else 20
```

#### 5.真值与假值

```python
# 如果一个对象计算为0，就是False
bool(0) # False
bool(0.0) # False

# 空字符串、空列表、空字典为False
bool('') # False
bool([]) # False
bool({}) # False

# None是False
bool(None) # False
```

#### 6.空值表示


| 数据结构 | 表示  |
| -------- | ----- |
| 空列表   | []    |
| 空字典   | {}    |
| 空集合   | set() |
| 空元组   | ()    |

#### 7.抑制print追加换行

```python
print(i, end='')
```

#### 8.for循环中获取index

```python
for index, item in enumerate(items):
    print(index, item)
```

#### 9.检测基本类型：type

```python
a = 100
b = 12.345
c = 'hello, world'
d = True
print(type(a))    # <class 'int'>
print(type(b))    # <class 'float'>
print(type(c))    # <class 'str'>
print(type(d))    # <class 'bool'>
```

#### 10.检测复杂类型：types

```python
>>> import types
>>> def fn():
...     pass
...
>>> type(fn)==types.FunctionType
True
>>> type(abs)==types.BuiltinFunctionType
True
>>> type(lambda x: x)==types.LambdaType
True
>>> type((x for x in range(10)))==types.GeneratorType
True
```

#### 11.检测class类型：isinstance（一个对象是否是该类型本身，或者位于该类型的父继承链上）

```python
>>> isinstance(h, Dog)
True

>>> isinstance([1, 2, 3], (list, tuple))
True
>>> isinstance((1, 2, 3), (list, tuple))
True
>>> isinstance('a', str)
True
>>> isinstance(123, int)
True
>>> isinstance(b'a', bytes)
True
```

#### 12.类型转换

```python
a = 100
b = 12.345
c = 'hello, world'
d = True
# 整数转成浮点数
print(float(a))    # 100.0
# 浮点型转成字符串 (输出字符串时不会看到引号哟)
print(str(b))      # 12.345
# 字符串转成布尔型 (有内容的字符串都会变成True)
print(bool(c))     # True
# 布尔型转成整数 (True会转成1，False会转成0)
print(int(d))      # 1
# 将整数变成对应的字符 (97刚好对应字符表中的字母a)
print(chr(97))     # a
# 将字符转成整数 (Python中字符和字符串表示法相同)
print(ord('a'))    # 97
```

#### 13.生成特定范围内的数字：range

```python
range(101)：可以用来产生0到100范围的整数，需要注意的是取不到101。
range(1, 101)：可以用来产生1到100范围的整数，相当于前面是闭区间后面是开区间。
range(1, 101, 2)：可以用来产生1到100的奇数，其中2是步长，即每次递增的值。
range(100, 0, -2)：可以用来产生100到1的偶数，其中-2是步长，即每次递减的值。
```

#### 14.反转list

```python
list[::-1]
```

#### 15.完美去重(去重的同时，保持顺序)

```python
def dedupe(items, key=None):
    seen = set()
    for item in items:
        val = item if key is None else key(item)
        if val not in seen:
            yield item
            seen.add(val)

>>> a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
>>> list(dedupe(a, key=lambda d: (d['x'],d['y'])))
[{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
>>> list(dedupe(a, key=lambda d: d['x']))
[{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
```

#### 16.开平方的方法

```python
16**0.5
>>> 4.0

import math
math.sqrt(16)
>>> 4.0
```
