---
title: 星号操作符
date: 2023-02-01 13:57:19
categories: Python
tags:
- python
permalink: python-unpacking-operators
---
### 一、*符号
#### 1.解包所有可迭代对象，比如list、tuple、string
```python
num_list = [1, 2, 3, 4, 5]
num_list_2 = [6, 7, 8, 9, 10]
new_list = [*num_list, *num_list_2]
print(new_list)  # [1,2,3,4,5,6,7,8,9,10]
```
<!--more-->

#### 2.剩余赋值
```python
first, *middle, last = 'ma'
print(first, middle, last)  # m [] a
```

#### 3.打包
```python
*names, = 'Michael', 'John', 'Nancy'
print(names)  # ['Michael', 'John', 'Nancy']
```

#### 4.函数传参
```python
def names_tuple(*args):
    return args
names_tuple('Michael', 'John', 'Nancy')
# ('Michael', 'John', 'Nancy')
```

### 二、**符号
#### 1.函数传参
```python
def names_dict(**kwargs):
    return kwargs
names_dict(Jane = 'Doe')
# {'Jane': 'Doe'}
names_dict(Jane = 'Doe', John = 'Smith')
# {'Jane': 'Doe', 'John': 'Smith'}
```

#### 2.合并字典
```python
num_dict = {'a': 1, 'b': 2, 'c': 3}
num_dict_2 = {'a': 4, 'e': 5, 'f': 6}
new_dict = {**num_dict, **num_dict_2}
print(new_dict)  # {'a': 4, 'b': 2, 'c': 3, 'e': 5, 'f': 6}
```

参考文章：https://towardsdatascience.com/unpacking-operators-in-python-306ae44cd480
