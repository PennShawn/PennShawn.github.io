---
title: Redis的一些命令
categories:
- 技术
tags:
- Java
- redis
---


### `SETEX key seconds value`
将键key的值设置为value,并将key的生存时间设置为seconds秒.如果key已经存在,将覆盖旧值.

### `SETNX key value`
当键key不存在时,将key的值设为value,返回1
当key存在时,不做任何动作,返回0

### `KEYS pattern`
查找所有符合给定模式的

### `INCR key`
为键key存储的数字值加一.
如果key不存在,那么会先初始化key为0,然后再执行`INCR`命令.
如果key存储的值不能被解释为数字,呢么会返回一个错误.

### `RPUSH key value [value...]`
将一个或多个value按从左到右的顺序依此插入到key的表尾

### `ZCARD key`
当key存在且是有序集时,返回有序集的基数,即元素的个数

### `LTRIM key start stop`
对一个列表进行修剪,就是说,让列表只保留指定区间的内容,不在指定区间的元素都将被删除.</br>
举个例子:</br>
执行命令`LTRIM list 0 2`, 表示只保留列表`list`的前三个元素,其余元素全部删除.</br>
你也可以使用负数下标,以`-1`表示列表的最后一个元素,`-2`表示列表的倒数第二个元素.</br>
`LTRIM`命令通常和`LPUSH`配合使用,如:</br>
```
LPUSH log newest_log
LTRIM log 0 99
```
这个例子模拟了一个日志程序,每次将最新的`newest_log`放到`log`列表中,并且只保留最新的100项.


### `GEOADD key longitude latitude member [longitude latitude member]`
将给定的 经度、纬度、名字添加到指定的key里面</br>
`GOEOADD`能够记录的坐标时有限的,非常接近两级的区域是无法被检索的.精确的坐标限制由 EPSG:900913 / EPSG:3785 / OSGEO:41001 等坐标系统定义.</br>有效的纬度范围介于-85..0112878度至 85.05112878 度之间。</br>
用法示例:
```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2

redis> GEODIST Sicily Palermo Catania
"166274.15156960039"

redis> GEORADIUS Sicily 15 37 100 km
1) "Catania"

redis> GEORADIUS Sicily 15 37 200 km
1) "Palermo"
2) "Catania"
```

### `GEORADIUSBYMEMBER key member radius m|km|ft|mi`
这个命令和`GRORADIUS`一样,可以找出位于指定范围的元素,但是`GEORADIUSBYMEMBER`的中心点由给定的元素位置决定.</br>
在游戏里面就可以根据这个找到玩家附近的玩家.
```
redis> GEOADD Sicily 13.583333 37.316667 "Agrigento"
(integer) 1

redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2

redis> GEORADIUSBYMEMBER Sicily Agrigento 100 km
1) "Agrigento"
2) "Palermo"
```

### `LPUSHX key value`
将`value`插入到`key`的表头,当且仅当`key`存在并且是一个列表.<br>
和`LPUSH key value [value...]`命令相反,当`key`不存在时,`LPUSHX`命令什么也不做.

### `TTL key`
`TTL`以秒为单位,返回当前key的剩余生存时间.`TTL(time to live)`.<br>
返回-1, 如果key没有到期超时。<br>
返回-2, 如果键不存在。

