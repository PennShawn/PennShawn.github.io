---
title: RedisSentinel搭建
categories: 
 - 技术
tags:
 - Java
 - redis
---


#### 1.安装redis
#### 配置主从复制
##### master的redis.conf
```
[root@localhost redis]# cd /app/redis   # 安装目录下新建conf和logs目录，存放文件，
[root@localhost redis]# mkdir conf      # conf和logs目录的创建，不是必须的，只
[root@localhost redis]# mkdir logs      # 是为了区别存放文件，增加辨识度
[root@localhost redis]# cp redis.conf ./conf/redis-M-6379.conf
[root@localhost redis]# cd conf
[root@localhost conf]# vi redis-M-6379.conf
修改内容如下：
     port 6379   # 要使用的端口
     daemonize yes
     pidfile /app/redis/conf/redis-6379.pid
     bind 192.168.50.101   # 本机IP
     logfile /app/redis/logs/redis-6379.log
     dbfilename dump-6379.rdb
     dir /app/redis/conf   # 数据文件存放位置
     appendonly no  #master

# 启动服务
[root@localhost bin]# ./redis-server /app/redis/conf/redis-M-6379.conf
[root@localhost bin]# ps -ef|grep redis
root      27556      1  0 20:31 ?        00:00:00 ./redis-server 192.168.50.101:6379              
root      27561  25602  0 20:31 pts/2    00:00:00 grep redis

[root@localhost bin]# ./redis-cli -h 192.168.50.101 -p 6379   #登录redis
192.168.50.101:6379> exit
```
##### slaver的redis.conf
```
[root@localhost redis]# cd /app/redis   # 安装目录下新建conf和logs目录，存放文件，
[root@localhost redis]# mkdir conf      # conf和logs目录的创建，不是必须的，只
[root@localhost redis]# mkdir logs      # 是为了区别存放文件，增加辨识度
[root@localhost redis]# cp redis.conf ./conf/redis-S-6379.conf
[root@localhost redis]# cd conf
[root@localhost conf]# vi redis-S-6379.conf
修改内容如下：
     port 6379   # 要使用的端口
     daemonize yes
     pidfile /app/redis/conf/redis-6379.pid
     bind 192.168.50.102   # 本机IP
     logfile /app/redis/logs/redis-6379.log
     dbfilename dump-6379.rdb
     dir /app/redis/conf
     appendonly yes #slave
     slaveof 192.168.50.101 6379  # master主的地址和端口

# 启动服务
[root@localhost bin]# ./redis-server /app/redis/conf/redis-S-6379.conf
[root@localhost bin]# ps -ef|grep redis
root      29164      1  0 20:43 ?        00:00:00 ./redis-server 192.168.50.102:6379              
root      29170  25317  0 20:43 pts/0    00:00:00 grep redis

[root@localhost bin]# ./redis-cli -h 192.168.50.102 -p 6379   #登录redis
192.168.50.102:6379> exit
```
##### 验证主从
```
# 主服务上的操作
[root@Centos01 bin]# ./redis-cli -h 192.168.50.101 -p 6379
192.168.50.101:6379> set name wsl
OK
192.168.50.101:6379> get name
"wsl"
192.168.50.101:6379> info

# 从服务上的操作
[root@Centos02 bin]# ./redis-cli -h 192.168.50.102 -p 6379
192.168.50.102:6379> get name
"wsl"
192.168.50.102:6379> info
```
#### 3.配置sentinel集群
在192.168.50.101 192.168.50.102机器上操作,两个sentinel配置一样
```
[root@localhost ~]# cd /app/redis-4.0.1
[root@localhost redis-4.0.1]# cp sentinel.conf /app/redis/conf/
[root@localhost redis-4.0.1]# cd /app/redis/conf
[root@localhost conf]# cp sentinel.conf sentinel-16379.conf
[root@localhost conf]# vi sentinel-16379.conf
# 改变内容如下：

protected-mode no
##sentinel实例之间的通讯端口  
port 16379  
##显示监控master节点192.168.50.101，master节点使用端口6379，最后一个数字表示投票需要的"最少法定人数"，
##比如有10个sentinal哨兵都在监控某一个master节点，如果需要至少6个哨兵发现master挂掉后，才认为master真
##正down掉，那么这里就配置为6，最小配置1台master，1台slave，在二个机器上都启动sentinal的情况下，哨兵
##数只有2个，如果一台机器物理挂掉，只剩一个sentinal能发现该问题，所以这里配置成1，至于mymaster只是一个
##名字，可以随便起，但要保证下面使用同一个名字
sentinel monitor mymaster 192.168.50.101 6379 1  
##表示如果10s内mymaster没响应，就认为SDOWN
sentinel down-after-milliseconds mymaster 10000  
##表示如果master重新选出来后，其它slave节点能同时并行从新master同步缓存的台数有多少个，显然该值越大，所有slave节
##点完成同步切换的整体速度越快，但如果此时正好有人在访问这些slave，可能造成读取失败，影响面会更广。最保定的设置
##为1，只同一时间，只能有一台干这件事，这样其它slave还能继续服务，但是所有slave全部完成缓存更新同步的进程将变慢。
sentinel parallel-syncs mymaster 1  
##表示如果15秒后,mysater仍没活过来，则启动failover，从剩下的slave中选一个升级为master
sentinel failover-timeout mymaster 15000    

# 启动服务
[root@localhost bin]# ./redis-sentinel /app/redis/conf/sentinel-16379.conf &
```


#### 本地验证
##### 1.启动master、slave和sentinel
```
 playcrab@shenpeng  ~  ps aux|grep redis
playcrab         24187   0.1  0.0  4305360    884 s006  SN   Mon02PM   1:50.51 redis-sentinel *:16379 [sentinel]
playcrab         22186   0.0  0.0  4295528    292 s008  S+   Mon11AM   0:00.03 redis-cli -p 6379
playcrab         22031   0.0  0.0  4305360    640   ??  Ss   Mon11AM   1:13.54 redis-server 127.0.0.1:6380
playcrab         21366   0.0  0.0  4306384    704   ??  Ss   Mon11AM   1:15.17 redis-server 127.0.0.1:6379
playcrab         52010   0.0  0.0  4295400    936 s007  S+   10:20PM   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn redis
```
##### 查看状态
6379
```
27.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=4853892,lag=1
master_repl_offset:4853892
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:3805317
repl_backlog_histlen:1048576
```
6380
```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:4856229
master_link_down_since_seconds:5
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```
##### kill掉maste的进程
6379挂了
```
127.0.0.1:6379> info replication
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```
6380没有立刻变成master
```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:4856229
master_link_down_since_seconds:5
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```
6380过一段时间后才变成master
```
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

##### 重新恢复6379后,6379变成slave
```
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:1672
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

