---
title: MongoDB分片
categories: 
 - 技术
tags:
 - Java
 - MongoDB
---

在比较大的访问量和数据量时,单个mongoDB压力会比较大,这时候就可以采用`分片`技术,将数据分在不同的机器上存储.
![image](http://note.youdao.com/yws/res/3508/DE7C3486F0AC4CDD85C588E7EC564DDA)
分片集群的构造:

 （1）mongos ：数据路由，和客户端打交道的模块。mongos本身没有任何数据，他也不知道该怎么处理这数据，去找config server

（2）config server：所有存、取数据的方式，所有shard节点的信息，分片功能的一些配置信息。可以理解为真实数据的元数据。

 （3）shard：真正的数据存储位置，以chunk为单位存数据。
 
 
Mongos本身并不持久化数据，Sharded cluster所有的元数据都会存储到Config Server，而用户的数据会分散存储到各个shard。Mongos启动后，会从配置服务器加载元数据，开始提供服务，将用户的请求正确路由到对应的碎片。

##### Mongos的路由功能:

　　当数据写入时，MongoDB Cluster根据分片键设计写入数据。

　　当外部语句发起数据查询时，MongoDB根据数据分布自动路由至指定节点返回数据。
