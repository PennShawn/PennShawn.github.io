---
title: Redis的SLOWLOG
categories:
 - 技术
tags:
 - Redis
---

`Slow log`是Redis用来记录查询执行时间的日志系统.它保存在内存里面,读写速度非常快,不必担心因为开始`slow log`而损害Redis的速度.

### 设置`SLOWLOG`
Slow log的行为有两个配置参数指定,可以通过改写redis.conf或者用`CONFIG GET`和`CONFIG SET`命令对它们动态的进行修改.<br>
<br>
第一个选项是`slowlog-log-slower-than`,它决定要对执行时间大于多少微秒的查询进行记录.(1秒=1000毫秒=1000000微秒).如:
```
CONFIG SET slowlog-log-slower-than 100
```
第二个选项是`slowlog-max-len`,它决定slow log最大能保存多少条日志.slow log本身是一个FIFO的队列,当队列大小超过`slowlog-max-len`时,最旧的一条日志将被删除.如:
```
CONFIG SET slowlog-max-len 1000
```

### 查看`SLOWLOG`
```
SLOWLOG GET [NUMBER]
```
查看number条日志
```
# 为测试需要，将 slowlog-log-slower-than 设成了 10 微秒

redis> SLOWLOG GET
1) 1) (integer) 12                      # 唯一性(unique)的日志标识符
   2) (integer) 1324097834              # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
   3) (integer) 16                      # 查询执行时间，以微秒为单位
   4) 1) "CONFIG"                       # 执行的命令，以数组的形式排列
      2) "GET"                          # 这里完整的命令是 CONFIG GET slowlog-log-slower-than
      3) "slowlog-log-slower-than"

2) 1) (integer) 11
   2) (integer) 1324097825
   3) (integer) 42
   4) 1) "CONFIG"
      2) "GET"
      3) "*"

3) 1) (integer) 10
   2) (integer) 1324097820
   3) (integer) 11
   4) 1) "CONFIG"
      2) "GET"
      3) "slowlog-log-slower-than"

# ...
```

### 查看当前日志的数量
```
SLOWLOG LEN
```

### 清空日志
```
SLOWLOG RESET
```
