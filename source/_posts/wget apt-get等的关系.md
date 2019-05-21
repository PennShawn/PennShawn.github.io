---
title: wget apt-get等的关系
categories :
- 技术
tags :
- linux
---

#### wget
wget类似于迅雷，是一种下载工具，

通过HTTP、HTTPS、FTP三个最常见的TCP/IP协议下载，并可以使用HTTP代理

 名字是World Wide Web”与“get”的结合。

 

 
#### yum
yum是redhat, centos 系统下的软件安装方式，基于Linux，

         全称为 Yellow dog Updater, Modified，

         是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器

         基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包。

 
#### rpm
rpm:  软件管理;   redhat的软件格式 rpm     r=redhat  p=package   m=management

             用于安装 卸载 .rpm软件

 

 

#### 串联下：

   使用wget下载一个 rpm包, 然后用 rpm -ivh  xxx.rpm  安装这个软件，嫌麻烦的话，就

   可以直接用  yum  install  sqoop   来自动下载和安装依赖的rpm软件。

 

 

ap-get是ubuntu下的一个软件安装方式，它是基于debain。




