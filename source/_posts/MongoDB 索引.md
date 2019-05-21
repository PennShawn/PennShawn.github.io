---
title: MongoDB 索引
categories :
- 技术
tags :
- Java
- MongoDB
---

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。

这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构

#####MongoDB使用 createIndex() 方法来创建索引。

createIndex()方法基本语法格式如下所示：
```
>db.collection.createIndex(keys, options)
```
语法中 Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。

实例
```
>db.col.createIndex({"title":1})
```
`createIndex()` 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引）。
```
>db.col.createIndex({"title":1,"description":-1})
```
options:可选参数，表示建立索引的设置。可选值如下：
```
background，Boolean，在后台建立索引，以便建立索引时不阻止其他数据库活动。默认值为false。
unique，Boolean，创建唯一索引。默认值 false。
name，String，指定索引的名称。如果未指定，MongoDB会生成一个索引字段的名称和排序顺序串联。
partialFilterExpression, document.如果指定,MongoDB只会给满足过滤表达式的记录建立索引.
sparse，Boolean，对文档中不存在的字段数据不启用索引。默认值是 false。
expireAfterSeconds,integer,指定索引的过期时间
storageEngine,document,允许用户配置索引的存储引擎
```
#####在后台创建索引的原因：
在前台创建索引期间会锁定数据库，会导致其它操作无法进行数据读写，在后台创建索引是，会定期释放写锁，从而保证其它操作的运行，但是后台操作会在耗时更长，尤其是在频繁进行写入的服务器上。

#####MongoDB提供的查看索引信息的方法：
1、查看集合索引
```
db.col.getIndexes()
```
2、查看集合索引大小
```
db.col.totalIndexSize()
```
3、删除集合所有索引
```
db.col.dropIndexes()
```
4、删除集合指定索引
```
db.col.dropIndex("索引名称")
```


#####Compound index
Mongodb 支持对多个 field 建立索引，称之为 compound index。Compound index 中 field 的顺序对索引的性能有至关重要的影响，比如索引 {userid:1, score:-1} 首先根据 userid 排序，然后再在每个 userid 中根据 score 排序。

创建 Compound index
在此创建一个 products collection：
```
{
 "_id": ObjectId(...),
 "item": "Banana",
 "category": ["food", "produce", "grocery"],
 "location": "4th Street Store",
 "stock": 4,
 "type": "cases"
}
```
复制代码然后创建一个 compound index：
`db.products.createIndex( { "item": 1, "stock": 1 } )`
复制代码这个 index 引用的 document 首先会根据 item 排序，然后在 每个 item 中，又会根据 stock 排序，以下语句都满足该索引：
```
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana", stock: { $gt: 5 } } )
```
条件 `{item: "Banana"} `满足是因为这个 query 满足 **prefix 原则**。
#####使用 compound 索引需要满足 prefix 原则
Index prefix 是指 index fields 的左前缀子集，考虑以下索引：
```
{ "item": 1, "location": 1, "stock": 1 }
```
这个索引包含以下 index prefix：
``{ item: 1 }
{ item: 1, location: 1 }
复制代码所以只要语句满足 index prefix 原则都是可以支持使用 compound index 的：
```
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana",location:"4th Street Store"} )
db.products.find( { item: "Banana",location:"4th Street Store",stock:4})
```
相反如果不满足 index prefix 则无法使用索引，比如以下 field 的查询：
```
the location field
the stock field
the location and stock fields
```
由于 index prefix 的存在，如果一个 collection 既有 {a:1, b:1} 索引 ，也有 {a:1} 索引，如果二者没有稀疏或者唯一性的要求，single index 是可以移除的。

参考：https://juejin.im/post/5ad1d2836fb9a028dd4eaae6
