---
title: MongoDB 查询文档
categories :
- 技术
tags :
- MongoDB
---

#### MongoDB 查询数据的语法格式如下：
```
db.collection.find(query, projection)
```

#### MongoDB常用的条件操作符:
```
$gt -------- greater than  >

$gte --------- gt equal  >=

$lt -------- less than  <

$lte --------- lt equal  <=

$ne ----------- not equal  !=

$eq  --------  equal  =
```

#### 一些其它用法:
```
查询 title 包含"教"字的文档：
db.col.find({title:/教/})

查询 title 字段以"教"字开头的文档：
db.col.find({title:/^教/})

查询 titl e字段以"教"字结尾的文档：
db.col.find({title:/教$/})
```

#### MongoDB AND 条件
MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。

语法格式如下：
```
db.col.find({key1:value1, key2:value2}).pretty()
```

#### MongoDB OR 条件
MongoDB OR 条件语句使用了关键字 $or,语法格式如下：
```
>db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

#### AND 和 OR 联合使用
以下实例演示了 AND 和 OR 联合使用，类似常规 SQL 语句为：` 'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'`
```
>db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()

{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```
#### skip(), limilt(), sort()三个放在一起执行的时候，执行的顺序是先 `sort()`, 然后是 `skip()`，最后是显示的`limit()`。

```
> db.kos.find({$or:[{"monKey":"monValue"},{"value":{$type:2},"name":"2"}]}).limit(1).skip(1).pretty()
```
显示第二条结果
