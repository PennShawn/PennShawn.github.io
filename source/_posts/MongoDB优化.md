---
title: MongoDB优化
categories: 
 - 技术
tags:
 - Java
 - MongoDB
---

#### 1.分析器
检测MongoDB分析器是否打开`db.getProfilingLevel()`
- `0` 表示没打开
- `1` 表示打开了,会记录超过了指定执行时间的语句.
- `2` 打开了,记录一切查询语句
```
> db.setProfilingLevel(1,200)
{ "was" : 0, "slowms" : 1, "ok" : 1 }
> db.getProfilingLevel()
1
> db.getProfilingStatus()
{ "was" : 1, "slowms" : 200 }
> db.setProfilingLevel(0)
{ "was" : 1, "slowms" : 200, "ok" : 1 }
```
打开了分析器之后,`show tables`就可以看到多了一个`system.profile`的文件,这里面记录的就是所有的被记录的语句的执行时间等信息.可以通过
`db.system.profile.find().pretty()`查看.

如果发现时间比较长，那么就需要作优化。比如:
- nscanned数很大，或者接近记录总数，那么可能没有用到索引查询。
- reslen很大，有可能返回没必要的字段。
- nreturned很大，那么有可能查询的时候没有加限制。
...

**`explain()`方法也能起到类似的分析功能**

#### 2.索引
索引的原理是建立指定字段的B+树,通过搜索B+树来找到指定Document的位置,所以如果要查询超过了一半的集合数据,不通过索引查询可能会快一点.

使用索引时还要主要索引的顺序,比如要查询name=jack和age=18的person,因为name重复的数量比较少,所以建立索引时,name在前面比较好.

在MongoDB建立索引能提高查询效率，但在MongoDB新增、修改效率上比较慢

参考: https://blog.csdn.net/xcy1193068639/article/details/95197495


