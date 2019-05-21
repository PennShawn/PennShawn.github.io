---
title: php执行shell命令
categories :
- 技术
tags :
- PHP
- linux
---


PHP提供共了多个专门的执行外部命令的函数：system()，exec()，passthru()。
 1. system()
　　原型：string system (string command [, int return_var])
　　system()函数很其它语言中的差不多，它执行给定的命令，输出和返回结果。第二个参数是可选的，用来得到命令执行后的状态码。
　　例子：
　　
　　system("/usr/local/bin/webalizer/webalizer");
　　?> 
 2. exec()
　　原型：string exec (string command [, string array [, int return_var]])
　 　exec()函数与system()类似，也执行给定的命令，但不输出结果，而是返回结果的最后一行。虽然它只返回命令结果的最后一行，但用第二个参数 array可以得到完整的结果，方法是把结果逐行追加到array的结尾处。所以如果array不是空的，在调用之前最好用unset()最它清掉。只有 指定了第二个参数时，才可以用第三个参数，用来取得命令执行的状态码。
　　例子：
　　
　　exec("/bin/ls -l");
　　exec("/bin/ls -l",  $res);
　  \#$res是一个数据，每个元素代表结果的一行
　　exec("/bin/ls -l", $res, $rc);
　　#$rc的值是命令/bin/ls -l的状态码。成功的情况下通常是0
　　?>
 3. passthru()
　　原型：void passthru (string command [, int return_var])
　 　passthru()只调用命令，不返回任何结果，但把命令的运行结果原样地直接输出到标准输出设备上。所以passthru()函数经常用来调用象 pbmplus（Unix下的一个处理图片的工具，输出二进制的原始图片的流）这样的程序。同样它也可以得到命令执行的状态码。
　　例子：
　　
　　header("Content-type: image/gif");
　　passthru("./ppmtogif hunte.ppm");
　　?>

.....

shell_exec函数:
```
$cmd = "curl -d \"data=%7b%22action%22%3a%22reload%22%7d\" http://10.2.145.23:62001/battle/admin";
$resp = shell_exec($cmd);
$json = json_decode($resp);
echo $json->code;
```

`json_decode()`能够将json字符串解析为json对象
