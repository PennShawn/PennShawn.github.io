---
title: ThreadLoacl
categories: 
 - 技术
tags:
 - Java
 - 多线程
---

ThreadLocal提供了线程独有的局部变量，可以在整个线程存活的过程中随时取用，极大地方便了一些逻辑的实现.



每个Thread内部都有一个ThreadLocalMap，我们调用ThreadLoacl的set方法的时候,其实是把这个ThreadLocal的弱引用当成keyset的值当成value存到ThreadLocalMap中。每当使用ThreadLocal的get(key)，寻找ThreadLocal实例的弱引用对应的value。


```
 public static void main(String[] args) {
        ThreadLocal<String> sLocal = new ThreadLocal<>();
        sLocal.set("a");
        ThreadLocal<String> sLocal2 = new ThreadLocal<>();
        sLocal2.set("b");
        
        //这时候ThreadLocalMap里就有两个<k,v>,分别用sLocal和sLocal2的弱引用当key.
        System.out.println(sLocal.get());//a
          System.out.println(sLocal2.get());//b
    }
```

#### ThreadLocalRandom
 Random方法的执行逻辑是,初始化一个seed,然后每次产生新的随机数时都会通过CAS将seed设置为新值,那么在多线程的情况下,就有可能自旋,降低性能.这时候就可以用ThreadLocalRandom来代替.调用`ThreadLocalRandom.current()`来改变当前线程的seed就不会有问题了.
 
##### 为什么key用弱引用
如果用的是强引用,我们在用完了一个ThreadLoca对象后,如果用threadLocal=null来销毁对象,但是由于ThreadLocalMap中的key还指向这个对象,threadLocal不会被回收,这就不符合我们threadLocal=null的业务逻辑了.
 
#### 内存泄漏问题
ThreadLocal对象最好在使用完后主动的remove,不然在垃圾回收时,key被回收成null了,value还在内存中,却获取不到了.尤其是在使用线程池的时候,线程会得到复用.
