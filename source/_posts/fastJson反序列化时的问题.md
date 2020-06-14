---
title: fastJson反序列化时的问题
categories: 
 - 技术
tags:
 - Java
---

fastJson在反序列化时,会优先尝试调用默认的无参构造函数,如果没有默认的构造函数,则会调用
相匹配的有参构造函数,如果有参构造函数里没有接受全部的参数,会导致确实的那一部分属性是默认值.
```
  public static void main(String[] args) {

        TestData data = new TestData("male");
        data.setName("lily");

        String s = JSONObject.toJSONString(data);
        data = JSONObject.parseObject(s, TestData.class);
        System.out.println(data.gender + " " + data.name); //male null
    }
```
如果只有一个构造函数:
```
 public TestData(String gender) {
        this.gender = gender;
    }
```
则,反序列化后得到的对象name值是null.

要解决这一点,加上无参的构造函数或者带全部的构造函数就行.
