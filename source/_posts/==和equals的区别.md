---
title: ==和equals的区别
categories: 
 - 技术
tags:
 - Java
---

对于基本数据类型,==和equals的功能是一样的

对于对象来说,== 比较的是它们在内存中的地址.

equals取决于它们的eauals方法.默认也是比较他们在内存中的地址
```
GhostHouse g1 = new GhostHouse();
        GhostHouse g2 = new GhostHouse();
        System.out.println(g1 == g2);       //false
        System.out.println(g1.equals(g2));  //false
```
像String会重写equals方法,这时候比较的就是它们的值是否相等.
```
 public boolean equals(Object anObject) {  
        if (this == anObject) {      //如果是同一个对象,必然返回true
            return true;
        }
        if (anObject instanceof String) {            
            String anotherString = (String)anObject;
            int n = value.length;
            
            //String用value这个char[]存值,这时候要比较value中每个字符是否相等
            if (n == anotherString.value.length) {     
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```


##### 为什么重写了equals方法就一定要重写hashCode方法?
这是一种公约:

equals相等,那么hashcode一定相等
equals不相等,hashcode有可能相等也有可能不相等
如果hashcode不相等,那么equals一定不相等.

还是上面GhostHouse的例子,如果重写了GhostHouse的equals方法,两个对象equals了,但是放进同一个set后,set大小为2,这就不符合常规了.


