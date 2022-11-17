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



