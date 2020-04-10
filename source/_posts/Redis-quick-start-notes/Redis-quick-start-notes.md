---
title: Redis-quick-start-notes
date: 2020-04-10 16:35:10
tags:
---

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
