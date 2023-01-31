---
title: collections模块
date: 2022-11-17 09:57:45
categories: Python
tags:
- python
permalink: python-collections-module
---

### 一.Counter计数

#### 1.查询某元素出现次数、查询出现此次最多的n个元素

```python
from collections import Counter

temp = Counter('abcdeabcdabcaba')
print(temp.items())  # dict_items([('a', 5), ('b', 4), ('c', 3), ('d', 2), ('e', 1)])

for item in temp.items():
    print(item)
"""
('a', 5)
('b', 4)
('c', 3)
('d', 2)
('e', 1)
"""

print(temp.most_common(1))  # 统计出现次数最多的一个元素,[('a', 5)]
print(temp.most_common(2))  # 统计出现次数最多个两个元素,[('a', 5), ('b', 4)]

print(temp['a'])  # 查询单个元素出现次数，5
temp['a'] += 1  # 增加单个元素出现次数，6
temp['a'] -= 1  # 增加单个元素出现次数，5
del temp['a']  # 删除,({'b': 4, 'c': 3, 'd': 2, 'e': 1})
```
<!--more-->

### 二、双端队列deque

#### 1.基本使用

```python
from collections import deque

>>> q = deque()
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3])
>>> q.appendleft(4)
>>> q
deque([4, 1, 2, 3])
>>> q.pop()
3
>>> q
deque([4, 1, 2])
>>> q.popleft()
4
```

#### 2.固定大小队列(超过则弹出)

```python
from collections import deque

q = deque(maxlen=3)
q.append(1)
q.append(2)
q.append(3)
q.append(4)
print(q)  # deque([2, 3, 4], maxlen=3)
```

### 三、defaultdict

```python
d = defaultdict(list)
for key, value in pairs:
    d[key].append(value)
```

### 四、命名元组namedtuple
#### 1.基本使用
```python
>>> from collections import namedtuple
>>> Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
>>> sub = Subscriber('jonesy@example.com', '2012-10-19')
>>> sub
Subscriber(addr='jonesy@example.com', joined='2012-10-19')
>>> sub.addr
'jonesy@example.com'
>>> sub.joined
'2012-10-19'
>>> len(sub)
2
>>> addr, joined = sub
>>> addr
'jonesy@example.com'
>>> joined
'2012-10-19'
```

#### 2.使用_replace()更新值
```python
>>> s = Stock('ACME', 100, 123.45)
>>> s
Stock(name='ACME', shares=100, price=123.45)
>>> s.shares = 75
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
>>>

>>> s = s._replace(shares=75)
>>> s
Stock(name='ACME', shares=75, price=123.45)
```

### 五、ChainMap
#### 1.定义
一个 ChainMap 接受多个字典并将它们在逻辑上变为一个字典。 然后，这些字典并不是真的合并在一起了， ChainMap 类只是在内部创建了一个容纳这些字典的列表 并重新定义了一些常见的字典操作来遍历这个列表

和字典update方法的区别：

a.如果出现重复键，那么第一次出现的映射值会被返回 

b.对于字典的更新或删除操作总是影响的是列表中第一个字典

c.原字典变了，会联动chainMap的字典

#### 2.基本使用
```python
from collections import ChainMap

a = {'x': 1, 'z': 3}
b = {'y': 2, 'z': 4}
c = ChainMap(a, b)
print(c['x'])  # Outputs 1 (from a)
print(c['y'])  # Outputs 2 (from b)
print(c['z'])  # Outputs 3 (from a)

# 修改
c['y'] = 10
print(a)  # {'x': 1, 'z': 3, 'y': 10}
```




