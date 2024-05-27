---
title: Python复习
date: 2024-05-10 09:58:23
categories: Python
tags:
- python
permalink: python-review
---
### 一、进程、线程与协程
#### 1.并发与并行
 - 并发：假的多任务，多个任务共用一个核，轮流占用时间片
 - 并行：真的多任务，多个核同时执行多个任务
<!--more-->

#### 2.阻塞与非阻塞
- 阻塞：程序在等待某个操作完成期间，自身无法继续干别的事情，则称该程序在该操作上是阻塞的。比如网络I/O阻塞、磁盘I/O阻塞
- 非阻塞：程序在等待某操作过程中，自身不被阻塞，可以继续运行干别的事情，则称该程序在该操作上是非阻塞的。

#### 3.CPU密集型与IO密集型任务
- CPU密集型：多进程
- IO密集型：协程

#### 4.区别
|          | 进程                                        | 线程                                      | 协程                       |
|----------|-------------------------------------------|-----------------------------------------|--------------------------|
| 概念       | 资源分配的最小单位，是线程的容器                          | 运算调度的最小单位                               | 用户态实现的运算调度单位，是可暂停和恢复的函数  |
| 独立资源     | 独立资源空间                                    | 多个线程共享父进程的资源                            | 由单个线程执行                  |
| 切换代价     | 高                                         | 中                                       | 低                        |
| 通信（同步）方式 | 消息队列Queue、管道、套接字                          | 进程全局变量、锁                                | Future，channel, pub/sub等 |
| 优点       | 1. 进程间相互独立，一个进程出了问题不会影响其它进程2. 可以利用多CPU的资源 | 1. 线程之间共享内存和变量，通信比较方便2.上下文切换资源消耗中等      | 1.没有锁2.切换开销小3.高并发        |
| 缺点       | 切换开销大，进程间通信复杂                             | 存在竞态条件，GIL锁，无法充分利用多核 | 不能使用多核资源                 |

### 二、asyncio
#### 1.概念
针对IO密集型任务，通过协程和事件循环实现的单线程并发编程。

#### 2.异步中执行同步操作
对于不支持await的同步io操作，可以在主线程中开启新的线程来处理
```python
def sleep(seconds):
    t = threading.current_thread()
    print('T同步耗时任务中的线程name:" : %s' % t.name)
    time.sleep(seconds)
    return f"睡了{seconds}s"

# 用于测试，在主线程中开启新的线程执行同步耗时任务
async def main():
    t = threading.current_thread()
    print('接受任务的Thread name : %s' % t.name)

    return await asyncio.to_thread(sleep, 3)
```

#### 3.同步中执行异步操作
##### (1)串行
```python
async def main(i):
    print(f'第{i}次运行任务')
    await asyncio.sleep(3)
    return "异步代码睡了3秒"

if __name__ == '__main__':
    start = time.time()

    # 异步执行1-串行
    for i in range(3):
        asyncio.run(main(i))
    print(time.time() - start) 

"""
第0次运行任务
第1次运行任务
第2次运行任务
9.00822901725769
"""
```
##### (2)并行（乱序）
```python
async def main(i):
    print(f'第{i}次运行任务')
    await asyncio.sleep(3)
    return "异步代码睡了3秒"

if __name__ == '__main__':
    start = time.time()

    # 异步执行2（将异步的部分并发执行）并行
    tasks = [main(i) for i in range(10)]
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.wait(tasks))
    print(time.time() - start)

"""
第8次运行任务
第1次运行任务
第9次运行任务
第4次运行任务
第5次运行任务
第3次运行任务
第6次运行任务
第2次运行任务
第0次运行任务
第7次运行任务
3.001689910888672
"""
```
##### (3)并行（顺序）
```python
async def main(i):
    print(f'第{i}次运行任务')
    await asyncio.sleep(3)
    return "异步代码睡了3秒"

if __name__ == '__main__':
    start = time.time()

    tasks = [main(i) for i in range(10)]
    loop = asyncio.get_event_loop()
    res = loop.run_until_complete(asyncio.gather(*tasks))

    print(res)

"""
第0次运行任务
第1次运行任务
第2次运行任务
第3次运行任务
第4次运行任务
第5次运行任务
第6次运行任务
第7次运行任务
第8次运行任务
第9次运行任务
['异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒', '异步代码睡了3秒']
"""
```
#### 4.`asyncio.gather`和`asyncio.wait`的区别
|          | gather                                        | wait                            |
|----------|-----------------------------------------------|---------------------------------|
| 返回值      | 任务的结果列表                                       | 两个集合，一个是已完成的任务集合，另一个是未完成的任务集合   |
| 结果是否有序   | 是                                             | 否                               |
| 是否支持任务取消 | 是                                             | 是                               |
| 超时处理     | 不支持设置超时,如果其中一个任务永远不返回,它将一直等待下去                |设置timeout参数,在指定时间后会取消未完成的任务|

### 三、魔法方法（Dunder/Magic Methods）
#### 1.概念
用双下划线包围的内置方法，通过重写这些方法，可以自定义类的行为和状态

#### 2.常用魔法方法
##### （1）`__new__`
在对象实例化时调用的第一个方法，用于创建实例对象。它接受类`cls`，然后将所有参数传递给`__init__`。需要返回一个对象实例，如果没有返回值，则实例化对象的结果为None

实现单例模式：
```python
# 单例模式
class Test:
    INS = None

    def __new__(cls, *args, **kwargs):
        if cls.INS is None:
            print("正在给Test创建对象")
            ins = super().__new__(cls, **kwargs)
            cls.INS = ins
            return ins
        else:
            return cls.INS


t1 = Test()
t2 = Test()
t3 = Test()
t4 = Test()

print(t1 is t2 is t3 is t4)  # True
print(id(t1))  # 4372691408
print(id(t2))  # 4372691408
print(id(t3))  # 4372691408
```
实现工厂模式：
```python
# 工厂模式
class A:
    pass


class B:
    pass


class C:
    pass


class Factory:
    """匹配工厂"""
    cls_map = {
        'A': A,
        'B': B,
        'C': C
    }

    def __new__(cls, value):
        target_cls = cls.cls_map.get(value)
        if target_cls is None:
            raise ValueError(f"不符合条件的value: {value}")
        return target_cls()


ins1 = Factory("A")
print(ins1.__class__.__name__)  # A

ins2 = Factory("D")
print(ins2.__class__.__name__)  # ValueError: 不符合条件的value: D

```

##### （2）`__init__`
初始化实例对象，为其添加属性
```python
class Test:
    def __new__(cls, *args, **kwargs):
        print("正在给Test创建对象")
        ins = super().__new__(cls, **kwargs)
        print(ins)  # <__main__.Test object at 0x102bcba60>
        return ins

    def __init__(self):
        print(self)  # <__main__.Test object at 0x102bcba60>
        self.age = 100
        self.name = 'xiaoming'
        print(self.name)  # xiaoming

    def func01(self):
        print(self.name)  # xiaoming
        print(self.age)  # 100


t = Test()
t.func01()

```
##### （3）`__enter__`和`__exit__`
上下文管理协议(with)的实现原理

执行顺序：
```python
class Test:
    def __init__(self):
        print('init')
        self.a = 111

    def __enter__(self):
        print('__enter__() is call!')
        return self

    def do_something(self):
        print('do something!')

    def __exit__(self, exc_type, exc_value, traceback):
        print('__exit__() is call!')
        print(f'type:{exc_type}')
        print(f'value:{exc_value}')
        print(f'trace:{traceback}')


with Test() as sample:
    sample.do_something()
    
"""
init
__enter__() is call!
do something!
__exit__() is call!
type:None
value:None
trace:None
"""

```

##### （4）`__aenter__`和`__aexit__`
周期任务：要求随主线程一起开始，随主线程一起结束

```python
import asyncio
import threading
from typing import List, NamedTuple, Coroutine, Callable


class PeriodTask(NamedTuple):
    on_stop: Callable[[], Coroutine]


class Runner:

    def __init__(self):
        print("开始初始化")
        self.scheduled_tasks: List[PeriodTask] = []
        self.status: bool = True

    def loop_when(self):
        return self.status

    def schedule_task(
            self,
            coro: Callable[[], Coroutine],
            loop_when: Callable[[], bool] = None,
            step: int = 2,
            on_stop: Callable[[], Coroutine] = None,
    ):
        async def task_body():
            print('后台任务运行的线程名为', threading.current_thread().name)
            while loop_when():
                await coro()
                await asyncio.sleep(step)
            if on_stop is not None:
                await on_stop()

        task = asyncio.create_task(task_body())
        self.scheduled_tasks.append(
            PeriodTask(
                on_stop=on_stop
            )
        )

    async def period_task_start(self):
        # print(threading.current_thread().name)
        print("我是一个周期任务-执行了")

    async def period_task_stop(self):
        print("我是一个周期任务-周期任务结束")

    async def __aenter__(self):
        self.schedule_task(self.period_task_start, self.loop_when, on_stop=self.period_task_stop)
        # print("主任务正在继续执行别的耗时任务",threading.current_thread().name)
        await asyncio.sleep(10)
        print('主任务睡了十秒后苏醒了')
        self.status = False

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        for task in self.scheduled_tasks:
            await task.on_stop()


async def run():
    async with Runner() as runner:
        pass


if __name__ == '__main__':
    asyncio.run(run())

```

##### （5）`__getattr__`和`__getattribute__`
`__getattr__`: 当访问一个不存在的属性时会被调用，常用于实现对缺失属性的回退机制，比如deprecated api信息的返回
```python
class MyClass:
    def __init__(self, name):
        self.name = name

    def __getattr__(self, attr):
        if attr == "age":
            return 30
        else:
            raise AttributeError(f"Attribute '{attr}' not found")

my_obj = MyClass("Bob")
print(my_obj.name)  # 输出：Bob
print(my_obj.age)  # 输出：30
print(my_obj.city)  # 输出：AttributeError: Attribute 'city' not found

```
`__getattribute__`: 每次访问属性时都会被调用，如果属性存在，则直接返回该属性，如果不存在，则走`__getattr__`。可用作属性拦截器、权限验证
```python
class Girl:

    def __init__(self):
        self.name = "小明"
        self.age = 17

    def __getattr__(self, item):
        return f"属性 {item} 不存在"

    def __getattribute__(self, item):
        if item == "age":  # 拦截器，作用：权限校验
            return "女人芳龄不可泄露，别问，问就是还不到 18 岁"
        return super().__getattribute__(item)


girl = Girl()
# name 属性存在，所以在 __getattribute__ 中直接返回
print(girl.name)
"""
小明
"""
# age 也是如此，也是在 __getattribute__ 中直接返回
# 只不过它相当于被拦截了
print(girl.age)
"""
女人芳龄不可泄露，别问，问就是还不到 18 岁
"""
# 父类在执行 __getattribute__ 的时候，发现 xxx 属性不存在
# 于是会触发 __getattr__ 的执行（如果没定义则抛出 AttributeError）
print(girl.xxx)
"""
属性 xxx 不存在
"""

```
注意：为了避免`__getattribute__`陷入无限循环，需要调用父类的`__getattribute__`
```python
super().__getattribute__(item)
```
##### （6）`__str__`和`__repr__`
`__str__`: 在调用str()、format()、print()函数时，生成一个友好易于阅读的输出形式

`__repr__`: 调用repr()或直接查看对象时，会调用该方法，主要用于调试和开发。当仅定义`__repr__`的时候， `__repr__` == `__str__`

##### （7）`__call__`
将类的实例对象变成可调用的函数。

实现自定义装饰器：
```python
import time


class Timer:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        start_time = time.time()
        result = self.func(*args, **kwargs)
        end_time = time.time()
        print('执行时间：{}秒'.format(end_time - start_time))
        return result


@Timer
def slow_func(n):
    time.sleep(n)
    return n


print(slow_func(2))
"""
执行时间：2.00334095954895秒
2
"""

```



### 四、OOP
#### 1.`@property`
##### （1）设置只读属性：
```python
class House:

    def __init__(self, price):
        self._price = price

    @property
    def price(self):
        return self._price

house = House(100)
print(house.price)  # 100
house.price = 200  # AttributeError: can't set attribute 'price'
```
##### （2）自定义属性修改/删除逻辑
```python
class House:

    def __init__(self, price):
        self._price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, new_price):
        if new_price > 0 and isinstance(new_price, float):
            self._price = new_price
        else:
            print("Please enter a valid price")

    @price.deleter
    def price(self):
        del self._price


house = House(100)
print(house.price)  # 100
house.price = 200  # Please enter a valid price
house.price = 200.0
print(house.price)  # 200.0
```

### 五、装饰器
#### 1.概念
装饰器是一个函数，入参是被装饰的函数，返回值是一个新函数，新函数会调用被装饰的函数，并维持被装饰函数的签名

#### 2.示例
##### （1）在旧功能不变的情况下，添加新功能:
```python
from functools import wraps


def verify_permissions(func):
    @wraps(func)  # 保留原函数的元信息
    def wrapper(*args, **kwargs):
        print("验证权限")
        return func(*args, **kwargs)

    return wrapper


@verify_permissions
def insert(data):
    print(insert.__name__)
    print(f"插入{data}")


insert("数据")
"""
验证权限
insert
插入数据
"""

```
##### （2）多层装饰器
加载/执行顺序：靠近main函数的装饰器先加载，远离main函数的装饰器功能先执行
```python
def dec1(func):  # func=wrapper002
    print("装饰器1正在加载中")

    def wrapper001(*args, **kwargs):
        print("装饰器1的功能正在执行中")
        func(*args, **kwargs)

    return wrapper001


def dec2(func):  # func=main
    print("装饰器2正在加载中")

    def wrapper002(*args, **kwargs):
        print("装饰器2的功能正在执行中")
        func(*args, **kwargs)

    return wrapper002


@dec1  # @dec1=dec1(func) ; func=dec2(main)
@dec2
def main():  # main=dec1(dec2(func))
    print("原始函数main函数执行了")


# 装饰器2正在加载中
# 装饰器1正在加载中
# 装饰器1的功能正在执行中
# 装饰器2的功能正在执行中
# 原始函数main函数执行了

if __name__ == '__main__':
    main()

```
##### （3）装饰器传参
```python
import asyncio
import functools


def i_am_decorator(
        func=None,
        first=None,
        second=None,
        third=None,
):
    if not all((first, second, third)):
        raise ValueError('参数缺失')

    if func is None:
        return functools.partial(i_am_decorator, first=first, second=second, third=third)

    print("执行新功能啦")

    if asyncio.iscoroutinefunction(func):
        async def wrapper(*args, **kwargs):
            return await func(*args, **kwargs)
    else:
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)

    return wrapper


# @i_am_decorator(first=True, second=True, third=True)
dec = i_am_decorator(first=True, second=True, third=True)


@dec  # main=dec(main)
def main():
    print('main执行了')


@dec
async def main1():
    print('main1执行了')


if __name__ == '__main__':
    main()
    asyncio.run(main1())

```

### 六、算法
#### 1.递归
##### （1）阶乘
```python
def fn(n):
    if n == 1:
        return 1

    return n * fn(n - 1)
```
##### （2）斐波那契数列
```python
# 1 1 2 3 5 8 13 21

def fibo(n):
    if n == 1 or n == 2:
        return 1
    else:
        return fibo(n-1) + fibo(n-2) 
```
##### （3）青蛙跳台阶
```python
"""
青蛙跳台阶，一次可以跳1层或2层，问跳上n级台阶，有多少种跳法？
"""
# 1 2 3 5 8 13 21 斐波那契数列

def climb_stairs(n: int) -> int:
    s = [1, 2]
    if n <= 2:
        return s[n - 1]
    while len(s) < n:
        s.append(s[-1] + s[-2])
    return s[-1]
```

##### （3）实现字符串的find方法
```python
"""
实现 find() 函数:
给你两个字符串 haystack 和 needle ，
请你在 haystack 字符串中找出 needle 字符串出现的第一个位置（下标从 0 开始）。
如果不存在，则返回 -1
"""
str1 = "haystack"
str2 = "ys"


def my_find(base_str, str_slice):
    base_str_len = len(base_str)
    str_slice_len = len(str_slice)
    if base_str_len < str_slice_len:
        return -1
    if base_str == str_slice:
        return 0
    for i in range(base_str_len - str_slice_len + 1):
        base_slice = base_str[i: i + str_slice_len]
        if base_slice == str_slice:
            return i
    return -1


print(my_find(str1, str2))

```

##### （4）二分法查找
```python
"""
使用二分法实现sqrt()函数，并返回整数部分
"""


def my_sqrt(x: int) -> int:
    l, r, ans = 0, x, -1
    while l <= r:
        mid = (l + r) // 2
        if mid * mid <= x:
            ans = mid
            l = mid + 1
        else:
            r = mid - 1
    return ans


print(my_sqrt(10))

```

### 七、python面试题
#### 1.Python 中的深拷贝和浅拷贝的区别并实现
```python
浅拷贝：
   	创建新的对象，但只拷贝了原始对象中的元素的引用
   	新对象中的元素和原始对象中的元素指向相同的内存地址
   	
   import copy	
   
   my_list = [1,2,[3,4]]
   copy_list = copy.copy(my_list)
   
   copy_list[2][0] = 99
   print(my_list) # [1,2,[99,4]]
   
   
深拷贝
   深拷贝创建新的对象，递归的复制原始对象及其所有的嵌套的对象的内容
   新的对象和原始对象及其所有的嵌套的对象都是独立的，新对象和原始对象互不影响
   
   
   import copy	
   
   my_list = [1,2,[3,4]]
   copy_list = copy.deepcopy(my_list)
   
   copy_list[2][0] = 99
   print(my_list) # [1,2,[3,4]]
```

#### 2.什么是迭代器（Iterator）和生成器（Generator）？
```python
迭代器Iterator
   迭代器是一种对象，用来遍历可迭代对象中的元素
   惰性，只在需要的时候才从可迭代对象中获取一个元素
   节省内存
   
   number = [1,2,3,4,5]
   iters = iter(number)
   
   print(next(iters)) #1
   print(next(iters)) #2
   
   
   
生成器Generator
   生成器是一种特殊的迭代器，通过函数来创建yield,生成器在每次调用时，执行到yield语句后，返回一个值，并退出函数执行，等待下一次调用，再次进入到上次退出的地方继续执行
   
   def even_number(n):
       for i in range(n):
           if i % 2 == 0:
               yield i
   
   evens = even_number(10)
   
   for num in evens:
       print(num)
```

#### 3.什么是 Python 中的 GIL ？如何解决？
```python
现象：使用CPython解释器时，同一时间只能有一个线程执行 Python 字节码，在多核处理器上也是如此，主要影响CPU密集型任务
目的：保证线程安全
解决方案：
    1.使用多进程
    2.使用其他python解释器
    3.使用异步编程，减少对线程的依赖
```

#### 4.请解释 TCP 和 UDP 协议的区别，并说明它们在网络通信中的不同应用场景
```python
TCP 是一种面向连接的、可靠的、具有流量控制和拥塞控制的协议，适合需要确保数据完整性的场景，比如网页浏览、电子邮件、文件传输
UDP 是一个无连接的、不保证可靠性的、没有流量控制和拥塞控制的协议，适合需要快速传输并能容忍部分数据丢失的场景，比如音视频传输、在线游戏、DNS
```
