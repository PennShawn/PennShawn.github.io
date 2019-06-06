---
title: javac命令找不到符号
categories: 
 - 技术
tags:
 - Java
---

今天在查看try-with-source的编译结果的时候,发现编译时报
```
TryWithSource.java:11: 错误: 找不到符号
        try (Connection connection = new Connection()) {
             ^
  符号:   类 Connection
  位置: 类 TryWithSource
TryWithSource.java:11: 错误: 找不到符号
        try (Connection connection = new Connection()) {
                                         ^
  符号:   类 Connection
  位置: 类 TryWithSource
2 个错误
```
目录结构如下:
```
-exception
    -trywithwource
        -Connection.java
        -Connection.class
        -TryWithSource.java
```
`TryWithSource`类依赖`Connection`,但是编译的时候找不到`Connection`.
```
public class TryWithSource {

    public static void main(String[] args) {
        try (Connection connection = new Connection()) {
            connection.sendData();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

将classpath设置在`trywithwource`目录也不行.后来检查`trywithwource`代码:
```
package exception.trywithsource;
```
原来是已经指明了包名是`exception.trywithsource`,当在`trywithsource`包下面执行`javac`时,在`trywithsource`并没有`exception.trywithsource`这个包,也就找不到`exception.trywithsource.Connection`,所以有三种解决方法:
1.切换到`exception`的上层目录
```
javac exception/tryWithSource/TryWithSource.java
```
就可以找到依赖的类了,
2.指定绝对路径
```
 javac -cp ~/code/JavaCode/MavenDemo/src/main/java/ TryWithSource.java
```
3.相对路径
```
javac -cp ../../ TryWithSource.java
```
当然`javac *.java`也可以.
