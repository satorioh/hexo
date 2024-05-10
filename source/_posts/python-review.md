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
