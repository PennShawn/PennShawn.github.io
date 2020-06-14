---
title: JVM的垃圾收集器
categories: 
 - 技术
tags:
 - Java
 - JVM
---

![image](http://note.youdao.com/yws/res/3994/80D37854DBA04F5399A5A6AB25C09B54)
```
Serial 复制算法 新生代 单线程 串行
ParNew 复制算法 新生代 多线程 并行
Parallel Scavenge 复制算法 新生代 多线程 并行
Serial Old 标记整理 老年代 单线程 串行
Parallel Old 标记整理 老年代 多线程 并行
CMS 标记清除 老年代 多线程 并发    


```

`-XX:+UseConcMarkSweepGC`使用`ParNew + CMS +Serial Old`组合进行垃圾回收.新生代用ParNew,老年代默认用CMS,当碎片信息太多的时候,触发Serial Old.

#### CMS(Concurrent Mark Sweep)
CMS是JVM第一款真正意义上的并发的垃圾收集器,垃圾回收线程和用户线程(基本上)并发工作,是一款以最短回收停顿时间为目标的垃圾收集器.

##### 它的工作流程分四个步骤:
1. 初始标记  暂停其它线程,记录下与root相连的对象   stop the word
2. 并发标记  同时开启GC和用户线程,记录可达对象
3. 重新标记  暂停用户线程,修正并发标记期间用户对对象可达性的改变 stop the world
4. 并发清除  同时开启GC和用户线程,并发清除标记对象

##### 缺点
- 无法处理浮动垃圾
- 由于是标记-清除算法,可能会造成空间碎片


##### 会出现两种问题
- promotion failure 新生代的survier区空间不够,新生代直接向老年代转移,但是老年代空间也不够,就会触发full GC了.<br>
解决方法有两种,1.增大survier区的空间大小;2.增大老年代空间大小,去掉suevier区.
- concurrent model failure: 老年代回收速度太慢了,导致还没回收完时,又有对象进入老年代,但老年代空间不够,这时候只能触发Full GC.<br>
避免这个问题可以调小-XX:CMSInitiatingOccupancyFraction参数的值.默认是92,即老年代被占用92%空间的时候,触发CMS.8%的空间作缓冲,可以讲缓冲区调大增加CMS次数,来避免Full GC.

#### G1收集器
![image](http://note.youdao.com/yws/res/4054/4E91D4C466CB4FF9A4283FDCF7E8DE3D)
G1的工作范围是整个堆,它保留了新生代和老年代的概念,但他们不再是相互隔离的,而是都有一个个Region组成,按逻辑上分区.

在最后的回收时,它会按Region的回收价值和成本排序,按照用户期望的GC停顿时间来安排回收.


##### 项目中的CMS调优
第一次测试的时候,ios1服频繁出现大停顿GC,基本一天一次,这是第二周才出现的问题,第一周的配置是:
```
-Xmx:90%内存
-XX:+UseCMSInitiatingOccupancyOnly 
-XX:CMSInitiatingOccupancyFraction=80
```
第二周因为内存占了90%机器总是报警,并且20%的缓冲感觉有点浪费,新的配置是:
```
-Xmx:70%内存
-NewSize:20%Xmx
去掉了-XX:CMSInitiatingOccupancyFraction=80
```
要减少concurrent mode failure，就基本需要让CMS的缓冲空间大于新生代的空间。
解决办法就是减少新生代大小，扩大老年代的缓冲空间大小。
iOS1服机器的内存是48G内存。
使用-Xmx85%，40.8G
新生代-Xmn=12.5%Xmx，5.1G
因此老年代剩余87.5%Xmx，35.7G。
设置-XX:CMSInitiatingOccupancyFraction=80，80%触发CMS GC，也就是28.56G。
所以缓冲区的大小就是7.14G，是大于新生代大小的，即使是将新生代所有内存一次性提升到老年代，也能放得下，不会出现空间不足。
