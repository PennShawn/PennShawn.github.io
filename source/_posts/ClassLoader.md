---
title: ClassLoader
categories: 
 - 技术
tags:
 - Java
---

##### ClassLoader的作用
JVM不是一开始就把所有的类都加载到内存中的,而是在第一次需要用到某个类的时候,才把它的字节码文件加载成Class对象加到内存中.这个过程需要CLassLoader来实现.

##### 字节码来源
- jar包中的.class文件
- 编译后的.class文件
- 远程服务器提供的字节流
- ...

#### 分类
##### AppClassLoader
加载当前应用的classpath下的类.parent属性是ExtensionClassLoader,构造时传入
##### ExtensionClassLoader
加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件.parent属性是null,这种情况下默认的父加载器是BootStrapClassLoader.
##### BootStrapClassLoader
最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等.由C++实现,是虚拟机的一部分,不是一个java类.

#### 主要方法
##### loadClass()
加载目标类的入口,先判断当前类加载器和父加载器是否已经加载过目标类,如果没有加载过会让父加载器尝试加载,如果父加载器都加载不了则调用`findClass()`方法自己加载.
##### findClass()
不同的类加载器将用不同的逻辑来获取目标类的字节码
##### defineClass()
拿到字节码之后,通过defineClass()来将字节码转换成Class对象.

##### 双亲委派模型
一个类加载器发现需要加载一个类时,先判断这个类有没有被加载过,如果没有,则将委托传递给父加载器,一直递归直到传给BootStrapClassLoader.如果BootStrapClassLoader无法加载目标类的话,则把委托向下传递,直到一个类加载器可以加载目标类.
