---
title: 启动apache时报错
categories :
- 技术
tags :
- PHP
---

```
playcrab@shenpeng  /etc/apache2  sudo apachectl restart
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using shenpeng.local. Set the 'ServerName' directive globally to suppress this message
```
查看网上解决方案
https://apple.stackexchange.com/questions/280099/apache-on-macos-sierra-ah00557-httpd-apr-sockaddr-info-get-failed-for-macbo

```
It's because you have two different installations of Apache - the one that came with the system and the one you installed yourself.
                                      – Allan May 26 '17 at 14:06
```

