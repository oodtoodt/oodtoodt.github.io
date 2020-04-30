---
title: Redis-quick-start-notes
date: 2020-04-10 16:35:10
tags:
- DB
- Redis
- company

categories:
- DB
- Redis
- documentation

---

。
<!--more-->

## 
二进制安全的字符串
lists
sets
sorted sets
hashes
bit arrays
hyperLogLogs
streams

INCR、DECR，atomic，原子增减操作
get、set、getset设置并返回原值，mset设置多个值，mget亦然
exists、del、type

## Redis expires
可以在时间到了之后删除key
可以用秒或者毫秒
解析都是1毫秒的
保存了过期的时间在硬盘上
可以以后缀的形式来在create
px和pttl是毫秒的形式

## Redis List
是用的Linked Lists，即链表实现的，更看重插入删除的效率。
如果需要大量的查询操作，你可以用另一种叫做sorted set的数据结构。
可以在恒定的时间内以恒定的长度获取到整个lists

可以用LPUSH来向一个list的头上加元素，RPUSH相反，在尾巴上加元素

LRANGE会提取整个列表（从头到尾）,需要两个参数，第一个元素和最后一个元素的位置（负数可以从最后开始）另外没有RRANGE

lpop,rpop

ltrim可以设定新的list的范围

### blocking operations
是为了解决polling(轮询)的问题而生的，polling有这样的问题：
强迫Redis和客户端处理无用命令，之后面临着处理更多无用调用还是等待更长的延迟的问题。
BRPOP和BLPOP：如果列表为空，它们将能够阻止，仅当新元素添加到列表中或者用户指定的超时时间到达时，它们才回返回到调用方。
可以指定多个list，返回第一个有响应的
BRPOP返回值和RPOP略有不同，它是包含了两个元素的数组，超时返回null
客户端以有序方式提供服务，第一个block的先被served
有兴趣可以查询RPOPLPUSH和BRPOPLPUSH，前者是更安全的队列，后者是阻塞变体

### 自动创建和删除key
指无需为添加删除元素之前是否有key（比如某个list）而操心。比如说c++或java，使用前要判断是否为空等等。适用于所有多个元素组成的数据类型。
注意，其中的唯一例外是流数据不会在删除元素时自动摧毁某个键

当然，类型还是要相同的。
### Redis Hashes
可以理解成为HashMap
hmset、hget、hmget(hget的array版)、hgetall、hincrby(指定增量)，hvals
值得注意的是，小散列（即一些具有较小值的元素）以特殊方式在内存中进行了编码，从而使它们具有很高的存储效率。
### Redis Sets
sadd添加
smembers查询，sismember
集合非常适合表示对象之间的关系。 例如，我们可以轻松地使用集合来实现标签。

对这个问题进行建模的一种简单方法是为我们要标记的每个对象设置一个集合。 该集合包含与对象关联的标签的ID。
```java
>sadd news:1000:tags 1 2 5 77
//逆关系：用给定标签标记所有新闻
>sadd tag:1:news 1000
```

我们可能需要包含标签1、2、10和27的所有对象的列表(注意，必须全部包含):
```java
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```
sunionstore执行多个集合之间的联合，并将结果存储到另一个集合中
scard指集合的基数
srandmember可以不remove随机取元素（还有取重复/非重复的元素的功能[?)

### Sorted sets
理解为正map...就是会依照value给key排序的set
zadd(注意是可以更新的，返回值0更新，1新增)
zrange、zrevrange WITHSCCORES、zrangebyscore、zremrangebyscore(范围内有多少个)
sorted-sets有个次排序资格：字典序，并且可以依照字典范围操作，有
```java
zrangebylex h [a [b
//注意[和(的问题
//非常重要，因为可以用作通用索引
```
### Bitmaps
不是实际的数据类型，而是在String类型上定义的一组面向位的操作。
位操作分为两类：固定时间的单个位操作；对位组的操作，例如计算给定范围内设置的位的数量
优点是存储信息时节省大量空间
setbit、getbit(超出范围始终视为0)
bitop执行按位运算
bitcount执行填充计数，报1的位数
bitpos第一个1的位置(of a string)
bitpos和bitcount都可以在字符串的字节范围内运行，而不是整个长度上运行

bitmaps的常用情景是
各种实时分析
存储于对象ID相关的空间高效且高性能的布尔信息

比如说记录网站用户每天访问。每次访问都可以按一个bitset的1，这样每个用户就都有一个字符串，bitcount就可以获得给定用户访问网站的天数，只需几个bitpos调用，或调用和分析bitmap client-side，就可以轻松计算出最长连续访问网站的人是谁。
【个人理解，就是循环跑bitpos计数。对于0很多的情况会很快。
可以用每个key存储M位的方式分割bitmap，使用bit-number MOD M即可。bit-number/M就可以访问。

## HyperLogLogs
>本质是一种算法，引申为一种数据结构。
>是一个*基数估计*算法，比如说一个集合｛0, 1, 3, 3, 4, 5}，其基数是5，而个数是6。因为3重复出现了两次。基数是个去重统计。通常用来统计一个集合中不重复的元素个数，例如统计某个网站的UV，或者用户搜索网站的关键词数量。数据分析、网络监控及数据库优化等领域都会涉及到基数计数的需求。
>其空间效率非常高。

HyperLogLog是一种概率数据结构，用于对唯一事物(技术上讲就是集合的基数)进行计数。它以内存为代价来交换精度。您只需要使用恒定数量的内存，使得误差小于1%。
尽管技术上HLL是不同的数据结构，Redis仍然视其为string，：可以用get、set等指令。
概念上使用HLL就像使用set。但HLL不是真正添加元素进去，因为数据结构仅含有一种不含实际元素的状态，它的API：
看到新元素，用PFADD加入。
想要检索pfadd添加过的数目的当前近似值时
一个例子是每天计算用户在搜索表单中执行的查询（唯一）

### 其他重要特征
+ 可以逐步迭代大型集合的键空间
+ 可以在服务器端运行lua脚本以改善延迟和带宽
+ Redis也是一个Pub-Sub服务器

