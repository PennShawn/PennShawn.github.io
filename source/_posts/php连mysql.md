---
title: php连mysql
categories :
- 技术
tags :
- PHP
- Mysql
---

从官网下载mysal并配置环境变量后
```
 ✘ playcrab@shenpeng  ~  mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
```
此时用php连mysql
```
<?php
$servername="127.0.0.1";
$username="root";
$pwd="12345678";
$conn = new mysqli($servername,$username,$pwd);
if ($conn->connect_error){
    die( "连接失败: ".$conn->connect_error);
}
echo "连接成功";
?>
```
会报错:
```
Warning: mysqli::__construct(): (HY000/2054): The server requested authentication method unknown to the client in /Users/playcrab/phpcode/PhpTest/src/MysqlConnect.php on line 11
连接失败: The server requested authentication method unknown to the client
```
发生这种错误，是由于MySQL8默认使用了新的密码验证插件：caching_sha2_password，而之前的PHP版本中所带的mysqlnd无法支持这种验证。
如果不能升级PHP，可以在MySQL 8中创建（或修改）使用caching_sha2_password插件的账户，使之使用mysql_native_password，这样先前版本的PHP就可以连接使用了。

在CREATE USER时，使用`IDENTIFIED WITH xxx_plugin BY 'password'`，比如：
```
CREATE USER 'native'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password!2#4';
```
使用ALTER USER修改已有账户的验证插件：
```
ALTER USER 'native'@'localhost' IDENTIFIED WITH mysql_native_password
```
或
```
ALTER USER 'native'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password';
```
采用前一种方式，账户的密码将被清除；BY子句将为账户设置新的密码。

/etc/my.cnf配置文件中，有一行：
```
# default-authentication-plugin=mysql_native_password
```
请删除注释符号“#”并重新启动mysqld使之生效，此后创建的账户均默认使用`mysql_native_password`。

如果您完成MySQL Server的安装之后，在没有启动过mysqld服务的情况下修改/etc/my.cnf配置，那么启动mysqld之后创建的'root'@'localhost'账户也是使用mysql_native_password插件的。

参考来源:https://www.cnblogs.com/cndavidwang/p/9357684.html

------

servername填`localhost`时,还会报错:
```
Warning: mysqli::__construct(): (HY000/2002): No such file or directory in /Users/playcrab/phpcode/PhpTest/src/MysqlConnect.php on line 11
连接失败: No such file or directory
```

问题出现的原因：

当主机填写为localhost时MySQL会采用 unix domain socket连接，当主机填写为127.0.0.1时MySQL会采用TCP/IP的方式连接。使用Unix socket的连接比TCP/IP的连接更加快速与安全。这是MySQL连接的特性，可以参考官方文档的说明4.2.2. Connecting to the MySQL Server：
```
On Unix, MySQL programs treat the host name localhost specially, in a way that is 
likely different from what you expect compared to other network-based programs. 
For connections to localhost, MySQL programs attempt to connect to the local server 
by using a Unix socket file. This occurs even if a --port or -P option is given to 
specify a port number. To ensure that the client makes a TCP/IP connection to the 
local server, use --host or -h to specify a host name value of 127.0.0.1, or the IP 
address or name of the local server. You can also specify the connection protocol 
explicitly, even for localhost, by using the --protocol=TCP option.
```
这个问题有以下几种解决方法：

1.使用TCP/IP代替Unix socket。即在连接的时候将localhost换成127.0.0.1。

2.修改MySQL的配置文件my.cnf，指定mysql.socket的位置：
/var/lib/mysql/mysql.sock (你的mysql.socket路径)。

3.直接在php建立连接的时候指定my.socket的位置（官方文档：mysqli_connect）。比如：
`$db = new MySQLi('localhost', 'root', 'root', 'my_db', '3306', '/var/run/mysqld/mysqld.sock')`

参考:https://segmentfault.com/q/1010000000328531
