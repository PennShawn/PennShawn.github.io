---
title: JVM对象头的组成
categories: 
 - 技术
tags:
 - Java
 - JVM
---

Java对象在内存中一般由三部分组成:
- 对象头
- 实例数据
- 对齐填充

#### JVM中对象头结构:
- Mark Word 
- Class Pointer 类型指针,指向类的元数据
- Array Length (数组对象才有)

#### Mark Word
Mark Word主要用来存储对象运行时的数据,如hashCode、GC年龄和锁信息等.
![image](http://note.youdao.com/yws/res/4142/9173C332DFB54335B3B8A49AD57CE13F)
比如,用4位表示GC分代年龄,一般对象在新生代经历了16次GC也就会晋升到老年代.
