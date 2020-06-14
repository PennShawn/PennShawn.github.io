---
title: idea打包项目成jar包
categories: 
 - 技术
tags:
 - Java
 - idea
---

在测试热更的时候,需要打包module的jar包.并且指定MANIFECT.MF文件,这个过程有一下的一些步骤
##### 1.配置
pom.xml: 在pom.xml中加入相应代码，让我们能在打包时生成Manifest。
```
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-jar-plugin</artifactId>
<configuration>
<archive>
<manifestFile>src/META-INF/MANIFEST.MF</manifestFile>
<manifest>
<addClasspath>true</addClasspath>
<classpathPrefix>lib/</classpathPrefix>
<mainClass>classloader.Main</mainClass>
</manifest>
</archive>
</configuration>
</plugin>

```
MANIFEST.MF: Manifest的作用其实就是指定Agent类是谁，从而能够定位到premain方法。
```
Manifest-Version: 1.0
Premain-Class: classloader.HotswapAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Can-Set-Native-Method-Prefix: true
```
##### 2.Artifact
![image](http://note.youdao.com/yws/res/4250/56FB6C4F752748BC83ABC6895B3E0CFF)
<br>artifact -> jar From modules with dependencies...
<br>然后选择对应的Module和主类,下面MANIFEST.MF的路径不要写.../main/java,似乎是idea的bug,要换个目录比如.../main/resource,不然之后对MANIFEST.MF修改都无效.
![image](http://note.youdao.com/yws/res/4255/81FF23B508BB44BDB11820EC603123E4)
<br>之后指定output文件,然后<br>
![image](http://note.youdao.com/yws/res/4262/B0551AD4BB7044BDA353BD825CE78EAC)
<br>build -》 Build Artifacts
最后就可以在output文件夹下找到.jar文件了.
