---
title: redis 知识总结(1)
date: 2018-11-09 23:33:43
tags:
---

`redis`是最常用的内存数据库，性能强悍，且支持丰富的数据类型。
一般主要用于高频访问数据的缓存，是现代构建高性能web系统的重要组件之一。

本文主要是总结和记录redis基本的命令和功能。

<!--more-->

### redis 命令

`redis`支持丰富的数据类型，本人使用最多的是: 字符串(string)，散列表(hash)，列表(list), 集合(set)，有序集合(sorted set)。
此外，`redis`还支持bitmap, hyperloglogs, 和地理数据(geospatial)等高级数据类型。
除了数据类型，`redis`还提供了通用功能的命令，维护和管理的命令。

下面针对各种命令作下基本的介绍：

* 键`keys`

redis中保存的数据都是以键值对的形式保存的。
本类命令主要包含redis较通用的功能，如：删除(`DEL`)，判断存在性(`EXISTS`), 标记过期(`EXPIRE`), 查询TTL(`TTL`), 迁移(`MOVE`)，重命名(`RENAME`)，类型检测(`TYPE`)等

更多参见文档:  http://www.redis.cn/commands/del.html

* 字符串(`string`)

字符串类型的值可以是字符或者数据。
redis支持直接对字符串类型修改以及追加: `SET`, `APPEND`; 若值为数字类型还可以进行增减操作(用于计数器): `INCR`, `INCRBY`, `DECR`, `DECRBY`等；
对于字符串类型的值还支持位运算和按位索引, 具体使用参见文档: http://www.redis.cn/commands/append.html

使用redis的字符串类型，可以很方便地实现一个分布式锁(仅用于演示，用于生产环境仍有风险！)。

```python
import redis

class Lock:
    def __init__(self, lock_key, timeout=3):
        self._lock_key = lock_key
        self._timeout = timeout
        self._get_locked = False

    def __enter__(self):
        _lock = redis.setnx(self._lock_key, 1)
        if _lock:
            self._get_locked = True
            redis.expire(self._lock_key, self._timeout)  # 注意：此处失败的话，锁无法移除；
            return True
        return False

    def __exit__(self, *exc_info):
        if self._get_locked:
            redis.delete(self._lock_key)


lock_key = "get_user_lock?user_id=%s" % 101
with Lock(lock_key) as lock:
    if not lock:
        return -1  # get lock failed!
    ...

```

* 列表(`List`)和集合(`Set`)

列表和集合都是常规的数据结构，redis也提供了丰富的命令支持多种操作；

列表命令: http://www.redis.cn/commands.html#list
列表提供了`安全队列`的命令，可以从一个列表(队列)取出元素返回并且加入另一个列表,而且这一系列操作是原子的；

集合类型支持常规的集合操作(增加元素，移除元素，取交、并、差集), 还支持对两个集合操作后结果保存在新的集合中(无需返回结果)；
集合操作: http://www.redis.cn/commands.html#set

* 有序集合(`Sorted Set`)

有序集合是redis特别提供的数据类型，有序集合除了想一般的集合一样可以保证每一个成员(`member`)是唯一的，还允许未成员指定一个分数(`score`);

通过对成员的分数做排序，这样就可以按照分值对有序的成员做批量操作；如批量获取，批量移除，取分数最大或者分数最小等等；
此外，有序集合还可以兼容一般的集合运算；

有序集合的概念非常强大；本人平时使用较多的就是`zrange`和`zrangebyscore`按照成员排序位置索引或者分数来获取顺序元素；这样可以方便地通过redis来维护有序列表数据，从而避免了依靠关系数据库做查询和排序；

具体的文档: http://www.redis.cn/commands/zadd.html

* 哈希表(`hash`)

redis的哈希表就像一组`keys`类型的集合，对于一组关联的数据使用哈希表可以更加方便地去管理；
http://www.redis.cn/commands/hexists.html

除了以上最常用的数据结构外，redis还支持地理位置(`geospatial`)和hyperloglogs，发布订阅模式，以及最新的流（`stream`);

地理位置类型是支持对经纬度坐标点数据的相关索引查询，如按照半径和距离等；
而hyperloglogs则是概率算法的计数器，常用于大数据统计，结果有误差；主要特点是占用资源极少；
redis支持类似于消息队列的发布和订阅模式，但是redis的发布订阅是`fire-and-forget`——只推送一次且不保存状态, 不太适合用于消息处理要求较高的场景；

redis 5.0中新增加了流类型(`stream`), 类似于`kafka`的任务流模式，支持不停地把消息追加到流中；所有的对流的读取可以依靠消息的索引便宜来获取消息；
鉴于流模式是redis新增加的一种类型，且功能强大，后续可以跟进相关的实践应用；

### redis 事务

redis 的事务是依靠一组命令来实现的: `MULTI`, `EXEC`, `DISCARD`, `WATCH`以及`UNWATCH`

一个事务中可以执行多条命令，并且事务会确保所有的命令被顺序地执行，全部成功或全部失败。
redis中由`MULTI`开启一个事务，添加待执行的命令(此时没有执行),执行`EXEC`后，事务中的全部命令开始一起执行，全部成功或全部失败;
`DISCARD`用于清空已开启的事务队列，放弃本次事务的执行；

```
> MULTI  # 开启一个事物
OK
> INCR foo
QUEUED
> INCR bar
QUEUED
>EXEC  # 执行事务，并一起返回所有结果
1) (integer) 1
2) (integer) 2
```
redis事务中添加的命令，只有返回`QUEUED`时才算成功加入事务; 最新的客户端会在有命令添加报错时放弃事务的执行；
而事务中的所有命令都会顺序执行，就算中间又失败，后续命令仍然继续执行；

redis事务不支持回滚；

`WATCH`可以为事务加上乐观锁, 只有被`WATCH`执行监听后和`EXEC`执行前健值没有被改变过，事务才会成功执行；否则事务会取消执行；

```
> SET mykey 1
OK
> WATCH mykey
OK
> INCRBY mykey 5  # watch mykey后，在事务执行前做了改动
6
> MULTI
OK
> INCR mykey
QUEUED
> EXEC  # 事务执行失败
(nil)
```

此外，redis支持命令流水线(`PIPELINE`)。但不同于事务，流水线并不能保证全部命令的原子性和一致性；
事务中的每个命令都会发往服务器保存，在最后执行完毕后一起返回结果；
流水线是把所有的命令缓存在客户端，集中一起发送到服务端去执行，再最后一起返回；
事务是可以被包含在流水线中；

### redis 持久化

redis提供了不同级别的的数据持久化，主要是: `RDB`和`AOF`;

* RDB

RDB 是在指定的时间点对数据做快照备份，默认已`dump.rdb`文件保存；
因为rdb文件保存的是数据快照，所以其内容比较紧凑，可以很方便地进行远程备份保存和数据回复；
但是数据只能回复到快照备份的时间点，这就意味着会有备份后的数据丢失。

RDB在只做数据快照时会fork出一个子进程来完成；
而且rdb在回复大数据集方面更有优势；

* AOF

AOF持久化方式会记录每次对服务器写操作，当服务重启时会重新执行这些写操作记录来回复数据；
redis以追加的方式把每个写命令加到记录的文件末尾，并且会在后台触发重写以压缩和减小文件的大小；
相比于RDB来说，AOF可以保存更完整的数据；redis重启后也会优先载入AOF记录文件来恢复数据；

AOF支持不同的`fsync`策略来记录指令: 无fsync, 每秒fsync, 每次fsync；
一般来说每秒钟fsync一次对性能影响较小，发生事故时也只会丢一小部分数据（具体丢多少要根据故障类型和数据同步到磁盘的侧率而定）；

对于记录损坏的AOF文件可以用`redis-check-aof`来进行修复；
AOF的记录文件体积一般要大于RDB的快照文件，而开启AOF会对性能有一定的影响，回复数据时也会慢一些；但是AOF可以保留更完整的数据；

持久化相关的文档: http://www.redis.cn/topics/persistence.html

redis数据持久化的讨论: (client)write => (db)user space memory => kernel buffer => disk controller => physical media
http://oldblog.antirez.com/post/redis-persistence-demystified.html

### 最佳实践

redis best practice: https://redislabs.com/community/redis-best-practices/
