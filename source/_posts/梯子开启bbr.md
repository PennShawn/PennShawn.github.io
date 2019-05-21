---
title: 梯子开启bbr
categories:
- 技术
tags:
- 梯子
- linux

---

##### 什么是BBR?：

TCP BBR是谷歌出品的TCP拥塞控制算法。BBR目的是要尽量跑满带宽，并且尽量不要有排队的情况。BBR可以起到单边加速TCP连接的效果。

Google提交到Linux主线并发表在ACM queue期刊上的TCP-BBR拥塞控制算法。继承了Google“先在生产环境上部署，再开源和发论文”的研究传统。TCP-BBR已经再YouTube服务器和Google跨数据中心的内部广域网(B4)上部署。由此可见出该算法的前途。

TCP-BBR的目标就是最大化利用网络上瓶颈链路的带宽。一条网络链路就像一条水管，要想最大化利用这条水管，最好的办法就是给这跟水管灌满水。

BBR解决了两个问题：

在有一定丢包率的网络链路上充分利用带宽。非常适合高延迟，高带宽的网络链路。

降低网络链路上的buffer占用率，从而降低延迟。非常适合慢速接入网络的用户。
Google 在 2016年9月份开源了他们的优化网络拥堵算法BBR，最新版本的 Linux内核(4.9-rc8)中已经集成了该算法。

对于TCP单边加速，并非所有人都很熟悉，不过有另外一个大名鼎鼎的商业软件“锐速”，相信很多人都清楚。特别是对于使用国外服务器或者VPS的人来说，效果更佳。

BBR项目地址：

https://github.com/google/bbr

------------

升级内核，第一步首先是升级内核到支持BBR的版本：
#### 1.yum更新系统版本：

yum update

#### 2.查看系统版本：

[root@server ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
[root@server ~]# 
#### 3.安装elrepo并升级内核：

[root@server ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
[root@server ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
[root@server ~]# yum --enablerepo=elrepo-kernel install kernel-ml -y
#### 4.更新grub文件并重启系统：

[root@server ~]# egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
CentOS Linux 7 Rescue 8619ff5e1306499eac41c02d3b23868e (4.14.14-1.el7.elrepo.x86_64)
CentOS Linux (4.14.14-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-693.11.6.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-c73a5ccf3b8145c3a675b64c4c3ab1d4) 7 (Core)
[root@server ~]# grub2-set-default 0
[root@server ~]# reboot
#### 5.重启完成后查看内核是否已更换为4.14版本：

[root@server ~]# uname -r
4.14.14-1.el7.elrepo.x86_64
[root@server ~]#
#### 6.开启bbr：

[root@server ~]# vim /etc/sysctl.conf    # 在文件末尾添加如下内容
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
#### 7.加载系统参数：

[root@vultr ~]# sysctl -p
net.ipv6.conf.all.accept_ra = 2
net.ipv6.conf.eth0.accept_ra = 2
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
[root@vultr ~]#
如上，输出了我们添加的那两行配置代表正常。

#### 8.确定bbr已经成功开启：

[root@vultr ~]# sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
[root@vultr ~]# lsmod | grep bbr
tcp_bbr                20480  2 
[root@vultr ~]# 
输出内容如上，则表示bbr已经成功开启。



