---
title: Redis的SCAN命令
categories:
 - 技术
tags:
 - redis
 - linux
---

由于Redis时单线程的,因此在使用一些时间复杂度为O(N)的命令时要非常谨慎,可能一不小心就会阻塞进程,导致Redis出现卡顿.<br>
有时我们需要从Redis实例成千上万的key中找出特定前缀的key列表来处理数据,如果用`keys`命令列出所有满足要求的key,当Redis里面的key的数量特别大时,就会造成Redis服务卡顿.<br>
Redis为了解决这个问题,在2.8版本中加入了指令`SCAN`.<br>
#### `SCAN`命令的基本使用
`SCAN cursor [MATCH pattern] [COUNT count]`.<br>
当`SCAN`的游标参数被设置为0时,服务器将开始一次新的迭代,而当Redis服务器向用户返回值为0时,表示迭代已经结束
```
127.0.0.1:6379> keys *
1) "db_number"
2) "key1"
3) "myKey"
127.0.0.1:6379> scan 0 MATCH * COUNT 1
1) "2"
2) 1) "db_number"
127.0.0.1:6379> scan 2 MATCH * COUNT 1
1) "1"
2) 1) "myKey"
127.0.0.1:6379> scan 1 MATCH * COUNT 1
1) "3"
2) 1) "key1"
127.0.0.1:6379> scan 3 MATCH * COUNT 1
1) "0"
2) (empty list or set)

```
可以看到,`SCAN`命令返回的是一个包含两个元素的数组,第一个数组元素是用于再一次迭代的新游标.第二个元素则是一个数组,这个数组中包含了所有被迭代元素的筛选结果.
