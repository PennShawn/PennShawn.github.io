---
title: MongoDB 聚合(三) aggregate()
categories :
- 技术
tags :
- Mongo
---

MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。有点类似sql语句中的 count(*)。

aggregate() 方法的基本语法格式如下所示：
```
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```
##### 示例
集合中的数据如下：
```
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'runoob.com',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'runoob.com',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
},
```
现在我们通过以上集合计算每个作者所写的文章数，使用aggregate()计算结果如下：
```
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "runoob.com",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
```
以上实例类似sql语句：
` select by_user, count(*) from mycol group by by_user`

在上面的例子中，我们通过字段 by_user 字段对数据进行分组，并计算 by_user 字段相同值的总和。

下表展示了一些聚合的表达式:


|表达式	|描述	|实例|
| ------- | -------- | ------- |
|$sum	| 计算总和。	| db.mycol.aggregate([{\$group : {_id : "\$by_user", num_tutorial : {\$sum : "\$likes"}}}])|


| 水果        | 价格    |  数量  |
| --------   |-----   | ---- |
| 香蕉        | 2   |   5    |
| 苹果        | 1      |   6    |
| 草莓        | 1      |   7    |
