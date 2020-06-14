---
title: AtomicDouble反序列化问题
categories: 
 - 技术
tags:
 - Java
---

AtomicDouble的value是transient修饰的,所以在序列化时会丢失value属性.
```
public class AtomicDouble extends Number implements java.io.Serializable {
  private static final long serialVersionUID = 0L;

  private transient volatile long value;
```
项目中,世界boss的血量是AtomicDouble修饰的,反序列化时就出现过问题.只能单独加个double来储存值.

但是AtomicLong的value不是由transient修饰,就没有问题.
```
public static void main(String[] args) {
        Data data = new Data();
        data.getAtomicLong().set(100);
        data.getAtomicDouble().set(1.1);
        String s = JSONObject.toJSONString(data);
        Data data1 = JSONObject.parseObject(s, Data.class);
        System.out.println(data1.getAtomicLong().get());//100
        System.out.println(data1.getAtomicDouble().get());//0.0
    }
```
