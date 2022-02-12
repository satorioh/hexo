---
title: redis基础命令
date: 2021-08-16 17:07:04
categories: Redis
tags:
- redis
permalink: redis-basic
---
安装参考：https://cloud.tencent.com/developer/article/1606701

### 一、全局命令
1.列出所有键（大量键时，生产禁止使用）：keys *

2.查询有多少个键：dbsize

3.是否存在某个键（0否1是）：exists [key]

4.删除一个/多个键，返回成功删除的键的个数：del [key/keys]

5.为键设置过期时间（不存在返回0）：expire [key] seconds

6.查询键的过期时间：ttl [key]
<!--more-->

说明：
> 返回大于0的整数：键的剩余过期时间
> 
> -1: 键未设置过期时间
> 
> -2: 键不存在

7.为键设置过期时间点(秒级时间戳)：expire [key] [timestamp]

8.清除键设置的过期时间：persist [key]

9.查询键的类型，不存在返回none：type [key]

10.重命名键：rename/renamenx [oldName] [newName]

11.随机返回一个键：randomkey

12.渐进式遍历所有键：scan

13.清库：flushdb

### 二、字符串命令
使用场景：缓存、计数、共享session、限速

1.设置键：set [key] [value] [秒级过期时间] [毫秒级过期时间] [nx|xx]

说明：
> nx: 键必须不存在，才会设置成功，用于添加
> xx: 键必须存在，才会设置成功，用于更新
> 
2.setnx和set xx：意义同上
```shell
setnx hello world
set hello newWorld xx
```

3.获取键的值，不存在返回(nil)：get [key]

4.批量设置键值：
```shell
mset a 1 b 2 c 3 d 4
```

5.批量获取键值,不存在返回(nil)：
```shell
mget a b c d
```

6.计数/自增：incr [key]
>说明：
> 
>键值不是整数，返回错误
> 
>键值是整数，返回自增后结果
> 
>键不存在，值按0，返回结果1

7.自减：decr [key]

8.自增指定数字：incrby [key] [number]

9.自减指定数字：decrby [key] [number]

10.自增浮点数：incrbyfloat [key]

11.追加值：append [key] [value]

12.查询字符串长度：strlen [key]
>说明：中文一个字占3个字节
```shell
set chinese "世界"
strlen chinese // 6
```

13.设置完后，返回旧的值：getset [key] [value]
```shell
set name bill
getset name kate // 返回 bill
```

14.设置指定位置的字符：setrange [key] [index] [value]
```shell
set word test
setrange word 0 p
get word // "pest"
```

15.获取部分字符串：getrange [key] [start] [end]

### 三、哈希命令
1.设置值：hset [key] [field] [value]
```shell
hset user:1 name bill
```

2.获取值：hget [key] [field]

3.删除一个/多个field，返回成功删除的个数: hdel [key] [field1] [field2]...

4.查询某个key的field个数：hlen [key]

5.批量设置field: hmset [key] [field1] [value1] [field2] [value2]...
```shell
hmset user:1 name mike age 18 job worker
```

6.批量获取field: hmget [key] [field1] [field2]...
```shell
hmget user:1 name age job
```

7.判断field是否存在：hexists [key] [field]

8.获取某个key的所有field: hkeys [key]

9.获取某个key的所有value：hvals [key]

10.获取某个key的所有field-value: hgetall [key]

11.自增field: hincrby [key] [field]

12.浮点自增field: hincrbyfloat [key] [field]

13.计算value的字符串长度：hstrlen [key] [field]

### 四、列表命令
使用场景：消息队列、分页获取文章列表

#### 1.添加操作
a.从右侧插入一个/多个值：rpush [listname] [value1] [value2]...
```shell
rpush list a b c // a b c
```

b.从左侧插入一个/多个值： lpush  [listname] [value1] [value2]...
```shell
lpush list 0 1 2 // 2 1 0
```

c.在某个值之前/之后插入：linsert [listname] [before/after] [targetValue] [value]

#### 2.查找操作
a.从左到右获取列表所有元素：lrange [listname] [start] [end]
>说明：下标start和end会包含自身

b.获取指定索引下标的元素：lindex [listname] [index]

c.获取列表长度：llen [listname]

#### 3.删除操作
a.左侧弹出元素：lpop [listname]

b.右侧弹出元素： rpop [listname]

c.删除指定元素：lrem [listname] [count] [value]
>说明：
> 
>count > 0，从左到右，删除count个值等于value的元素
> 
>count < 0，从右到左，删除count个值等于value的元素
> 
>count=0，删除所有的值等于value的元素

d.按照索引范围修建列表：ltrim [listname] [start] [end]

#### 4.修改操作
a.修改指定下标的元素：lset [listname] [index] [newValue]

#### 5.阻塞操作
brpop和blpop: brpop [listname1] [listname2]...[timeout]
>说明：timeout单位为秒
> 
>如果list为空：则等待timeout秒后返回(timeout为0则一直等待)，如果此时有元素加入，则立即返回
> 
>如果list不为空：则立即返回
> 
>当list为多个时，会从左至右遍历，一旦有哪个list的元素能弹出，则立即返回
> 
>先请求的客户端会优先获取到返回值

### 五、集合(set)命令
使用场景：给用户贴标签、抽奖、社交喜好

与列表区别：无序、无重复，支持并交补差

#### 1.集合内操作
a.添加元素：sadd [key] [element1] [element2]...

b.删除元素：srem [key] [element1] [element2]...

c.计算元素个数：scard [key]

d.判断元素是否在集合中(0否1是)：sismember [key] [element]

e.随机从集合中返回指定个元素(默认1个)：srandmember [key] [count] 或 spop [key] [count]
说明：两者区别，spop会删除元素

f.获取集合的所有元素：smembers [key]

#### 2.集合间操作
a.求多个集合的交集：sinter [key1][key2]...

b.求多个集合的并集：sunion [key1][key2]...

c.求多个集合的差集：sdiff [key1][key2]...

d.将交并差的结果保存，结果也为set：sinterstore/sunionstore/sdiffstore [destKey] [sourceKey1][sourceKey2]...

### 六、有序集合(zset)命令
使用场景：排行榜、社交

与集合的区别：元素也不能重复，但用来排序的score可以重复

#### 1.集合内操作
a.添加元素：zadd [key] [score1] [element1] [score2] [element2]...

b.删除元素：zrem [key] [element1] [element2]...

c.计算元素个数：zcard [key]

d.获取元素的分数：zscore [key] [element]

e.获取元素的排名（从0位开始）: zrank/zrevrank [key] [element]
>说明：zrank从低到高排名，zrevrank从高到低

f.增加元素的分数：zincrby [key] [incrementScore] [element]

g.获取指定排名范围的元素（从0位开始）: zrange/zrevrange [key] [start] [end] [withscores]
>说明：start/end为排名，zrange从低到高，zrevrange从高到低；withscores时会同时返回分数

h.获取指定score范围的元素: zrangebyscore/zrevrangebyscore [key] [min] [max] [withscores]

i.获取指定score范围的元素个数：zcount [key] [min] [max]

j.删除指定排名范围的元素：zremrangebyrank [key] [start] [end]

k.删除指定score范围的元素：zremrangebyscore [key] [min] [max]

#### 2.集合间操作
a.求多个集合的交集：zinterstore [destKey] [numKeys] [key1][key2]...weights [weight] aggregate [sum/min/max]

b.求多个集合的并集：zunionstore [destKey] [numKeys] [key1][key2]...weights [weight] aggregate [sum/min/max]
