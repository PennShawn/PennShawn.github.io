---
title: MongoDB 聚合(一) group
categories :
- 技术
tags :
- MongoDB
---

MongoDB目前提供了三个可以执行聚合操作的命令：`aggregate`、`mapReduce`、`group`.

### 1.group
##### 参数说明：
 - `key`：用来分组文档的字段。

 - `initial`: 每组都分享一个”初始化函数“

 - `reduce`: 执行的reduce函数，第一个参数是当前的文档对象，第二个参数是上一次function操作的累计对象，有多少个文档， $reduce就会调用多少次。

 - `condition`：（可选）执行过滤的条件

 - `finalize`：（可选）在reduce执行完成，结果集返回之前对结果集最终执行的函数。

例如，我们按年级分组，查出每个年级的学生姓名：
````
db.students.group({
key:{grade:true},
initial:{stuNames:[], count:0},
$reduce:function(cur, prev){
prev.stuNames.push(cur.name);
}
})
```
如果只想查询age大于20的人，group有这么两个可选参数: `condition` 和`finalize`：
```
db.students.group({
key:{grade:true},
initial:{stuNames:[], count:0},
$reduce:function(cur, prev){
prev.stuNames.push(cur.name);
},
finalize:function(prev){
prev.count = prev.stuNames.length;
},
condition:{age:{$gt:20}}
})
```
```
#查询每个栏目最贵的商品价格, max()操作
{
  key:{cat_id:1},
  cond:{},
  reduce:function(curr , result) {
      if(curr.shop_price > result.max) {
          result.max = curr.shop_price;
      }
  },
  initial:{max:0}
}

#查询每个栏目下商品的平均价格
{
  key:{cat_id:1},
  cond:{},
  reduce:function(curr , result) {
      result.cnt += 1;
      result.sum += curr.shop_price;
  },
  initial:{sum:0,cnt:0},
  finalize:function(result) {
      result.avg = result.sum/result.cnt; //在每次分组完毕后进行运算
  }
}
```
##### group其实略微有点鸡肋,因为既然用到了mongodb,那复制集和分片是避无可免的,而group是不支持分片的运算

