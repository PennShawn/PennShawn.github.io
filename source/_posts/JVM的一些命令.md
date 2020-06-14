---
title: JVM的一些命令
categories: 
 - 技术
tags:
 - Java
 - JVM
---

##### jstack
`jstack PID` 可以查看进程的堆栈信息
```
playcrab@shenpeng  ~/KOs/kos   brNew_tk  jstack 38374
2020-03-16 16:35:54
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.201-b09 mixed mode):

"DetectiveRecord-Timer" #37 prio=5 os_prio=31 tid=0x00007ff8136b8800 nid=0x9307 runnable [0x000070000cc9c000]
   java.lang.Thread.State: RUNNABLE
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	- locked <0x0000000796871140> (a java.net.SocksSocketImpl)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at org.apache.http.conn.socket.PlainConnectionSocketFactory.connectSocket(PlainConnectionSocketFactory.java:75)
	at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:142)
	at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:359)
	at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:381)
	at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:237)
	at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:185)
	at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:89)
	at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:111)
	at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185)
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83)
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:108)
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56)
	at com.playcrab.kos.tools.HttpClientHelper.doPostJson(HttpClientHelper.java:111)
	at com.playcrab.kos.manager.DetectiveManager.pushPostErrMsg(DetectiveManager.java:169)
	at com.playcrab.kos.manager.DetectiveManager.consumeErrorsQueue(DetectiveManager.java:100)
	at com.playcrab.kos.manager.DetectiveManager$1.run(DetectiveManager.java:67)
	at java.util.TimerThread.mainLoop(Timer.java:555)
	at java.util.TimerThread.run(Timer.java:505)
```


##### jstat
`jstat -gc PID`会打印进程的gc信息
```
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
84480.0 103424.0 20210.3  0.0   364544.0 28478.2   494592.0   315254.1  48640.0 47337.8 5888.0 5570.2     16    1.306   4      2.217    3.524
```

##### jcmd
顾名思义,jcmd会向jvm发送一些指令,根据指令的不同,可以代替部分的jstat、jstack

```
jcmd PID Thread.print 命令等同于 jstack PID

# 等同 jmap -dump:live,format=b,file=FILE_WITH_PATH
jcmd PID GC.heap_dump FILE_WITH_PATH

# 等同 jmap -dump:format=b,file=FILE_WITH_PATH
jcmd PID GC.heap_dump -all FILE_WITH_PATH

# 等同 jmap -histo:live PID
jcmd PID GC.class_histogram

# 等同 jmap -histo PID
jcmd PID GC.class_histogram -all
```
##### jmap
jmap主要用于获取堆相关的信息

```
获取堆概要信息 jmap -heap PID

jmap -dump:format=b,file=FILE_WITH_PATH PID 将 JVM 的堆 dump 到指定文件，如果堆中对象较多，需要的时间会较长，子参数 format 只支持 b，即二进制格式

# 打印 JVM 堆中的类实例统计信息，以占用内存的大小排序，同样，如果 JVM 未响应命令，也可以使用 -F 参数
jmap -histo PID

# 也可以只统计堆中的存活对象，加上 live 子参数，但使用 -F 时不支持 live
jmap -histo:live PID
```


