---
title: linux文件属性
categories: 
 - 技术
tags:
 - linux
---

我们可以通过`ls -l`命令来显示当前目录下文件的属性
```
 playcrab@shenpeng  ~/KOs/meta  ls -l
total 24
-rw-------@ 1 playcrab  staff  6703 May 14 20:25 check.py
drwxr-xr-x  9 playcrab  staff   288 May 15 16:46 config
-rw-------@ 1 playcrab  staff   285 May 14 21:08 config.ini
drwxr-xr-x@ 7 playcrab  staff   224 May 15 16:46 dtonline
```

#### 第一栏 `drwxr-xr-x`
##### 文件类型
第一个字符代表这个文件是『目录、文件或链接文件等等』：

- 当为[ d ]则是目录，例如上表档名为『.gconf』的那一行；
- 当为[ - ]则是文件，例如上表档名为『install.log』那一行；
- 若是[ l ]则表示为连结档(link file)；
- 若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
- 若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

##### 不同用户对文件的权限
接下来的字符中，以三个为一组，且均为『rwx』 的三个参数的组合。分别表示`文件拥有者的权限`、`拥有者所属组的组员的权限`和`其他用户的权限`.<br>
1. 其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。 要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。
2. 还可以用数字来表三每组权限:r=4,w=2,x=1
   - rwx权限等同于 4+2+1= 7
   - rw-权限等同于 4+2= 6   
   - r-x权限等同于 4+1= 5
   - ...
 
##### 可以通过chmod命令来执行权限,如:
```
chmod ug=wx,o=x file
```
- u表示文件拥有者,g表示文件拥有者的群组(group),o表示其它以外的人,a表示三者都是.
- +表示增加权限,-表示减少权限,=表示指定权限.

#### 第二栏 
表示有多少档名连结到此节点(i-node)

#### 第三栏`playcrab`
表示文件的拥有者
##### 更换文件拥有者 chown
`chown [拥有者][:群组] file`
如
```
[root@localhost test6]# ll
---xr--r-- 1 root users 302108 11-30 08:39 linklog.log
---xr--r-- 1 root users 302108 11-30 08:39 log2012.log
-rw-r--r-- 1 root users     61 11-30 08:39 log2013.log
[root@localhost test6]# chown mail:mail log2012.log 
[root@localhost test6]# ll
---xr--r-- 1 root users 302108 11-30 08:39 linklog.log
---xr--r-- 1 mail mail  302108 11-30 08:39 log2012.log
-rw-r--r-- 1 root users     61 11-30 08:39 log2013.log
```

#### 第四栏 `staff`
表示文件的所属组
##### 更改文件所属组 chgrp
与chown类似,不过只更改群组
```
[root@localhost test]# ll

---xrw-r-- 1 root root 302108 11-13 06:03 log2012.log

[root@localhost test]# chgrp -v bin log2012.log

“log2012.log” 的所属组已更改为 bin

[root@localhost test]# ll

---xrw-r-- 1 root bin  302108 11-13 06:03 log2012.log
```
加上`-v`表示显示处理信息

#### 第五栏 6703 
表示文件的大小,默认为byte

#### 第六栏 `May 14 20:25`
表示这个文件的建档日期或者是最近的修改日期

#### 第七栏 `check.py`
表示这个文件的档名
如果以`.`开头表示是隐藏文件

