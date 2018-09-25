---
title: Redis简单讲解 
categories: Redis
tags: [redis]
date: 2018-7-25 23:01:41 
author: gravel
---




## Redis是什么

Redis是由意大利人[Salvatore Sanfilippo][1]（网名：antirez）开发的一款内存高速缓存数据库。Redis全称为：Remote Dictionary Server（远程数据服务），该软件使用C语言编写，Redis是一个key-value存储系统，它支持丰富的数据类型，如：string、list、set、zset(sorted set)、hash。


## Redis数据结构 

redis是一种高级的key:value存储系统，其中value支持五种数据类型：

1.字符串（string）
2.列表（list）
3.集合（set）
4.有序集合（sorted set）
5.哈希（hashes）

### String

String是Redis值的最基础的类型。Redis中使用的字符串是通过包装的，基于c语言字符数组实现的简单动态字符串(simple dynamic string, SDS)一个抽象数据结构。在大多数情况下，redis不能对string类型有什么进一步的操作。其基本操作命令有set、get、strlen、getrange、append：
```
SET key value
GET key
STRLEN key
GETRANGE  key start end
APPEND key value
```
在大多数情况之外，就是string中存储的为纯数字的情况，redis可以将字符串当做数字进行进一步操作，这些操作包括decr、decrby、incr、incrby和incrbyfloat。

### List
Redis的List类型其实就是每一个元素都是String类型的双向链表。使用list时,可以像对待栈一样使用pop和push操作，但是这个栈两端都能进行操作；也可以像对待数组一样使用一个index参数来操作。list的操作命令略杂，主要分为两类：L开头的和R开头的，L代表LEFT或者LIST，进行一些从列表左端进行的操作，或者一些与端无关的操作；R代表RIGHT，进行一些从列表右端进行的操作。常用命令有：

* lpush——在key对应的list的头部添加一个元素。

* lrange——获取key对应的list的指定下标范围的元素，-1表示获取所有元素。

* lpop——从key对应的list的尾部删除一个元素，并返回该元素。
* rpush——在key对应的list的尾部添加一个元素。

* rpop——从key对应的list的尾部删除一个元素，并返回该元素。

### Set

set用于存储一组不重复的值，类似List的一个列表，与List不同的是Set不能有重复的数据。基本操作有sadd和sismember：
```
SADD key member [member ...]
SISMEMBER key member
```
关于集合的操作有这些：
```
SINTER key [key ...] --取交集
SUNION key [key ...] --取并集
SDIFF key [key ...] --求差
```

### sorted set
SortSet顾名思义，是一个排好序的Set，它在Set的基础上增加了一个顺序属性score，这个属性在添加修改元素时可以指定，每次指定后，SortSet会自动重新按新的值排序。基本操作有zadd、zcount、zrank：
```
ZADD key score member [score member ...]
ZCOUNT key min max
ZRANK key member
```
### hash 

最后要给大家介绍的是hashes，即哈希。哈希是从redis-2.0.0版本之后才有的数据结构。Hash是一个String类型的field和value之间的映射表，即redis的Hash数据类型的key（hash表名称）对应的value实际的内部存储结构为一个HashMap，因此Hash特别适合存储对象。相对于把一个对象的每个属性存储为String类型，将整个对象存储在Hash类型中会占用更少内存。常用命令有：

* hset——设置key对应的HashMap中的field的value

* hget——获取key对应的HashMap中的field的value

* hgetall--获取key对应的hashMap中的所有属性


## redis持久化

redis提供了两种持久化的方式，分别是RDB（Redis DataBase）和AOF（Append Only File）。

### RDB
简而言之，就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上；RDB，是将redis某一时刻的数据持久化到磁盘中，是一种快照式的持久化方法。

redis在进行数据持久化的过程中，会先将数据写入到一个临时文件中，待持久化过程都结束了，才会用这个临时文件替换上次持久化好的文件。正是这种特性，让我们可以随时来进行备份，因为快照文件总是完整可用的。

对于RDB方式，redis会单独创建（fork）一个子进程来进行持久化，而主进程是不会进行任何IO操作的，这样就确保了redis极高的性能。

如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。
#### 缺点
如果你对数据的完整性非常敏感，那么RDB方式就不太适合你，因为即使你每5分钟都持久化一次，当redis故障时，仍然会有近5分钟的数据丢失。

### AOF
则是换了一个角度来实现持久化，那就是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了。

其实RDB和AOF两种方式也可以同时使用，在这种情况下，如果redis重启的话，则会优先采用AOF方式来进行数据恢复，这是因为AOF方式的数据恢复完整度更高。

如果你没有数据持久化的需求，也完全可以关闭RDB和AOF方式，这样的话，redis将变成一个纯内存数据库，就像memcache一样。
我们通过配置redis.conf中的appendonly yes就可以打开AOF功能。如果有写操作（如SET等），redis就会被追加到AOF文件的末尾。

默认的AOF持久化策略是每秒钟fsync一次（fsync是指把缓存中的写指令记录到磁盘中），因为在这种情况下，redis仍然可以保持很好的处理性能，即使redis故障，也只会丢失最近1秒钟的数据。
因为采用了追加方式，如果不做任何处理的话，AOF文件会变得越来越大，为此，redis提供了AOF文件重写（rewrite）机制，即当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，假如我们调用了100次INCR指令，在AOF文件中就要存储100条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET指令，这就是重写机制的原理。

#### 缺点

在同样数据规模的情况下，AOF文件要比RDB文件的体积大。而且，AOF方式的恢复速度也要慢于RDB方式。

### 应用场景
#### 排行榜应用，取TOP N操作
比如微信的跳一跳，根据得分你通常想要得到：

* 列出前50名高分选手

- 列出某用户当前的全国排名

这些操作对于Redis来说应该很简单，在每次用户产生新的分数时：
```
ZADD topScore  <score>  <username>
```
得到前50名高分用户：
```
ZREVRANGE topScore 0 49
```
用户的全球排名也相似，只需要：
```
ZRANK topScore <username>
```

###  简单实例展示
打开你的idea