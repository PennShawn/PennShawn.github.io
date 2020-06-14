---
title: synchronized
categories :
- 技术
tags :
- Java
- 多线程
---

synchronized可以分成5类用法,可以用来修饰 实例方法
-------------------------------
| 修饰的目标 | 实际锁住的对象 | 伪代码 |
| ----       | ---            | ---    |
| 实例方法   | 类的实例对象   |public synchronized void remove(){...    }
| 静态方法   | 类对象         |public static synchronized void remove(){...}|
| 实例对象   | 类的实例对象   |synchronized(this) {...}|
| class对象  | 类对象         |synchronized (RatedArena.class){...}
| 任意实例对象obj | 实例对象obj | String s = "";<br>synchronized (s){...} |

写了两个方法,一个遍历方法,一个remove方法.如果有并发问题,那么遍历方会会抛一个异常.
```
public  synchronized void remove() throws InterruptedException {
        System.out.println("remove start");
        list.remove(0);
        list.add(String.valueOf(KOSTimeUtils.getCurrentMills()));
        list.add(String.valueOf(KOSTimeUtils.getCurrentMills()));
        Thread.sleep(100);
        System.out.println("remove end");
    }

    public  synchronized void forLoop() throws InterruptedException {
        for (String s : list) {
            Thread.sleep(100);
            System.out.println(" loop   " + s);
        }
    }
```
然后用两个线程分别跑这两个方法.
```
 static List<String> list = Lists.newArrayList();

    @Test
    public void testSynchronizedMethod() throws InterruptedException {
        list.add(String.valueOf(KOSTimeUtils.getCurrentMills()));
        new Thread(new Runnable() {

            @Override
            public void run() {
                while (true) {
                    try {
                        SynchronizedMethodTest test = new SynchronizedMethodTest();
                        test.forLoop();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
        new Thread(new Runnable() {

            @Override
            public void run() {
                while (true) {
                    try {
                        SynchronizedMethodTest test = new SynchronizedMethodTest();
                        test.remove();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
        Thread.sleep(10000000L);
    }
```
##### 修饰静态方法肯定是没问题的
##### 修饰非静态方法如果都是用同一个对象的remove()和forLoo()方法也没问题,但是如果修改一下调用方法:
```
SynchronizedMethodTest test = new SynchronizedMethodTest();
test.forLoop();
...
SynchronizedMethodTest test = new SynchronizedMethodTest();
test.remove();
```
loop的时候就会抛异常了
```
java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at thread.SynchronizedMethodTest.forLoop(SynchronizedMethodTest.java:61)
	at thread.SynchronizedMethodTest$1.run(SynchronizedMethodTest.java:25)
	at java.lang.Thread.run(Thread.java:748)
```
##### 因为他们锁的是不同的类实例,但是list是静态的,操作的是同一个list.
这是由于synchronized是基于每个对象的monitor来实现的,不同对象不需要抢同一个monitor

##### 如果有一个方法没有被synchronized修饰,那就不需要抢monitor了,也会有并发问题

