---
title: final、finally、finalize各自的用法
categories: 
 - 技术
tags:
 - Java
---

首先, final、finally、finalize这三者出了名字相似,没有太多联系.

#### finally
finally 则是 Java 保证重点代码一定要被执行的一种机制,我们可以使用 try-finally 或者 catch-finally 来进行类似关闭 JDBC 连接、保证 unlock 锁等动作。<br>
对于需要关闭的连接等资源,如果其实现了`AutoCloseable`接口,就可以用try-with-source优雅的关闭资源.
```
try {
  // do something
  System.exit(1);
} finally{
  System.out.println(“Print from finally”);
}
```
是个特例,finally里面的语句不会被执行.
#### finalize
finalize 是一个不推荐使用的方法,在Java9中,已经被标记为标记为`deprecated`.因为你无法保证 finalize什么时候执行，执行的是否符合预期。使用不当会影响性能，导致程序死锁、挂起等。

#### final
final可以用来修饰类,方法,变量,分别有不同的意义.final修饰的类是不可继承的,final修饰的方法是不可重写的,final修饰的变量是不可修改的.<br>
final修饰的属性的初始化可以在编译期，也可以在运行期，初始化后不能被改变。对于基本类型数据，final会将值变为一个常数（创建后不能被修改）；但是对于对象句柄（亦可称作引用或者指针），final会将句柄变为一个常数（进行声明时，必须将句柄初始化到一个具体的对象。而且不能再将句柄指向另一个对象。因此final修饰的变量,基本数据类型不可变的是其内容，而引用数据类型不可变的是其引用,引用变量本身的行为不受final约束.
例如:
```
 final List<String> strList = new ArrayList<>();
 strList.add("Hello");
 strList.add("world");  
```
是没有问题的.而如果修改`strList`的引用,
```
strList = new ArrayList<>();
```
编译器会报错.<br>
如果需要对对象的行为进行约束,可以构造一个`Immutable`变量.guava中`ImmutableList、ImmutableMap`等得到的变量是`Immutable`的,对象的行为也会收到约束.如:
```
        ImmutableList<String> immutableList = ImmutableList.of("a", "b");
        ImmutableList<String> immutableList2 = ImmutableList.<String> builder().add("1").add("2")
                .build();
        immutableList.add("s");//java.lang.UnsupportedOperationException
        immutableList2.add("s");//java.lang.UnsupportedOperationException
```
对象构建完成之后,再改变的时候会抛`java.lang.UnsupportedOperationException`异常.<br>
 - 如果要实现 immutable 的类，我们需要做到： 将class自身设为final
 - 所有成员变量定义为private final 并且不要实现setter
 - 通常构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值。防止他输入对象被他人更改
 - 如果确实需要实现geter方法，或者其他返回内部状态的方法，使用copy-on-write原则,创建私有的copy.
 
#### static
static可以修饰：属性，方法，代码段，内部类（静态内部类或嵌套内部类).
static修饰的属性的初始化在编译期（类加载的时候），初始化后能改变.<br>
被static修饰的成员变量或成员方法独立于该类的任何对象,也就是说,他不依赖于特定的实例,只要这个类被加载,Java虚拟机就可以根据类名在运行时数据区的方法区内找到他们.

参考:https://time.geekbang.org/column/article/6906
     https://www.jianshu.com/p/3e9811667132
