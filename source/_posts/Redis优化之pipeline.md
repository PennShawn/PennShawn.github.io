---
title: Redis优化之pipeline
categories: 
 - 技术
tags:
 - java
 - redis
---

Redis官网有一篇文章专门提到了通过使用Pipelining来提升Redis处理性.https://redis.io/topics/pipelining. Pipeline主要是从RTT的角度区提高性能的.
##### RTT(Round Trip Time)带来的问题
Redis是一个基于TCP的请求响应式服务器,通常情况下,一个Redis指令首先是客户端发送请求到Redis服务器,并监听Socket返回,通常是以阻塞的方式等待服务端返回.服务端处理完请求后,将结果返回给客户端.这样一个来回所需要的时间就是RTT.我们业务服务器如果和Redis不再同一个机器上时,中间的通信还受网络延迟的影响.

ping一下数据库的服务器,虽然ttl只有2毫秒,但在客户端是单线程处理的情况下,每秒钟只能和数据库产生500次请求/响应,而Redis对外宣称的性能可是每秒几万笔的处理能力.这种情况下,RTT就成了瓶颈了.

##### Pipeline的使用场景
一般我们在风格呢更新一个排行榜或者批量的Redis数据的时候,都是在一个循环里面发送多次Redis请求.Pipeline就可以将这些请求打包一起发给Redis了.
```
Pipeline p = jedis.pipelined();
        for (int i = 0; i < 1000; ++i) {
            p.set("key" + i, "a");
        }
        List<Object> results = p.syncAndReturnAll();
        jedis.close();
```
在请求量小的时候Pipeline的效果并不明显,但像根据排行榜批量该数据,全服玩法批量添加报名玩家等,就可以用到Pipeline了.

##### 可能的问题
```
IMPORTANT NOTE: While the client sends commands using pipelining, the server will be forced to queue the replies, using memory. So if you need to send a lot of commands with pipelining, it is better to send them as batches having a reasonable number, for instance 10k commands, read the replies, and then send another 10k commands again, and so forth. The speed will be nearly the same, but the additional memory used will be at max the amount needed to queue the replies for this 10k commands.
```
使用Pipeline时,Redis会用一个缓存队列来缓存处理结果,这样是很消耗内存的.所以一次也不要打包太多的命令,比如10k的命令,收到返回后再传10k的命令,两次请求和一次请求的RTT差距几乎是可以忽略的,但是占用内存就少了.

