---
title: OrderedDict和dict区别
date: 2022-11-17 10:24:01
categories: Python
tags:
- python
permalink: python-dict-ordereddict-difference
---

> Some differences from dict still remain:

* The regular dict was designed to be very good at mapping operations. Tracking insertion order was secondary.
* The OrderedDict was designed to be good at reordering operations. Space efficiency, iteration speed, and the performance of update operations were secondary.
* The OrderedDict algorithm can handle frequent reordering operations better than dict. As shown in the recipes below, this makes it suitable for implementing various kinds of LRU caches.
* The equality operation for OrderedDict checks for matching order.<!--more-->
  A regular dict can emulate the order sensitive equality test with `p==q and all==k2 for k1,k2 in zip(p,q))`.
* The `popitem()` method of OrderedDict has a different signature. It accepts an optional argument to specify which item is popped.
  A regular dict can emulate OrderedDict’s `od.popitem(last=True)` with `d.popitem()` which is guaranteed to pop the rightmost (last) item.
  A regular dict can emulate OrderedDict’s `od.popitem(last=False)` with `(k:=next(iter(d)),d.pop(k))` which will return and remove the leftmost (first) item if it exists.
* OrderedDict has a `move_to_end()` method to efficiently reposition an element to an endpoint.
  A regular dict can emulate OrderedDict’s `od.move_to_end(k,last=True)` with `d[k]=d.pop(k)` which will move the key and its associated value to the rightmost (last) position.
  A regular dict does not have an efficient equivalent for OrderedDict’s `od.move_to_end(k,last=False)` which moves the key and its associated value to the leftmost (first) position.
* Until Python 3.8, dict lacked a `__reversed__()` method.

参考链接：
[dict-ordereddict-difference](https://docs.python.org/3/library/collections.html#ordereddict-objects)

