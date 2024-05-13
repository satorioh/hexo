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

##### （4）`__getattr__`和`__getattribute__`
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
##### （5）`__str__`和`__repr__`
`__str__`: 在调用str()、format()、print()函数时，生成一个友好易于阅读的输出形式

`__repr__`: 调用repr()或直接查看对象时，会调用该方法，主要用于调试和开发。当仅定义`__repr__`的时候， `__repr__` == `__str__`

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
