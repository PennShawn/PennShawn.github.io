---
title: Mongo和Redis的备份与恢复
categories: 
 - 技术
tags:
 - MongoDB
 - redis
---

测试提了一个需求是需要备份两个服的Mongo和Redis数据,然后对两个服进行合服,测试完之后,再恢复成原来的数据,方便下次测试.所以,需要先将Redis和Mongo备份一次,测试完成后再恢复回来.

#### Mongo备份与恢复
##### Mongo备份
`mongodump -h dbhost -d dbname -o dbdirectory`
- -h 指定备份的数据库
- -o 指定需要备份到的文件夹

##### Mongo恢复
可以直接热恢复
`mongorestore -d test2 dump/test2  --drop`
- -d 指定要恢复的数据库,后面可以指定备份文件的文件夹
- --drop 恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除.
```
playcrab@shenpeng  ~  mongorestore -d test3 dump/test2  --drop
2020-07-07T15:00:00.695+0800	building a list of collections to restore from dump/test2 dir
2020-07-07T15:00:00.741+0800	reading metadata for test3.test2 from dump/test2/test2.metadata.json
2020-07-07T15:00:00.774+0800	restoring test3.test2 from dump/test2/test2.bson
2020-07-07T15:00:00.777+0800	restoring indexes for collection test3.test2 from metadata
2020-07-07T15:00:00.778+0800	finished restoring test3.test2 (1 document)
2020-07-07T15:00:00.778+0800	done
```

#### Redis备份与恢复
##### Redis备份
我们可以主动的让Redis生成一份快照
`bgsave`
查看状态
```
 ✘ xxxx@shenpeng  ~ redis-cli info |grep rdb
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1594090507
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
```
bgsave完成后,要注意给生成的dump.rdb文件改个名字,防止redis继续生成dump文件覆盖了备份数据.
```
127.0.0.1:6379> CONFIG GET *
  1) "dbfilename"
  2) "dump.rdb"
  ...
  125) "appendonly"
  126) "no"
  127) "dir"
  128) "/Users/playcrab"
```
可以查看redis恢复时的文件夹和文件名,测试完后,就可以恢复了.
##### redis恢复
恢复时,关闭aof持久化,然后将备份的rdb覆盖上一步文件夹中的rdb文件,启动redis-server就行了.
