---
title: 梯子KCP加速
categories:
- 技术
tags:
- 梯子
- linux

---
####下载
首先是下载工具,服务器端和客户端的版本最好一样.
客户端:https://github.com/xtaci/kcptun/releases
服务器:
`mkdir kcptun`
` cd kcptun`

`wget https://github.com/xtaci/kcptun/releases/download/v20170904/kcptun-linux-amd64-20170904.tar.gz`

`tar zxvf kcptun-linux-amd64-20170904.tar.gz`

`rm -f kcptun-linux-amd64-20170904.tar.gz`

####服务端开启kcptun
```
[root@vultr kcptun]# ./server_linux_amd64 -l :4001 -t 127.0.0.1:8381 -key password1 --mode "fast2" --log 4001.log &
[1] 10723
[root@vultr kcptun]# tail -100f 4001.log
```
####客户端
如果ss可以支持kcptun,那么不用下载别的客户端了,直接在ss上配置.
地址端口是服务器kcptun的地址和端口,加密类型和mode别填错了.

如果不支持,需要下载和服务器相同版本的kcptun客户端.
在客户端可执行文件的同一文件夹里新建client.json
```
{
	"localaddr": "127.0.0.1:4001",
	"remoteaddr": "45.32.125.148:4001",
	"key": "password1",
	"crypt": "aes",
	"mode": "fast2",
	"conn": 1,
	"mtu": 1350,
	"sndwnd": 512,
	"rcvwnd": 512,
	"nocomp": false
}
```
新建start.sh
```
client_darwin_amd64 -c client.json
```
双击start.sh
或者`client_windows_amd64.exe -c client.json`
