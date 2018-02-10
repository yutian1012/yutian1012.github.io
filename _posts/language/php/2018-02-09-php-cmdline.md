---
title: php命令行
tags: [php]
---

PHP的命令行模式能使得PHP脚本能完全独立于WEB服务器单独运行。

```
# 查看帮助
php -h
```

1）查看php的安装版本信息

```
# 切换到php的解压目录（安装目录）
cd E:\work\installer\php-7.0.27
# 查看版本号
php -v
```

2）命令行运行php文件

```
# php运行脚本文件
php -f "hello.php"
```

3）php运行代码

```
# 运行php代码
php -r "echo 'helloworld';"
```

4）php接收参数

```
# 传递参数，使用空格分隔参数
# 全局变量$argc表示参数的个数，全局变量$argv存储参数内容，
# $argv下标为0的元素为脚本名称，从下标为1处开始存储参数值
# 输出结果为3 helloworld
php -r "echo $argc;echo ' ';if($argc>1)echo $argv[1]; " heloworld welcome
```

5）php在linux下创建脚本文件

如果使用Unix系统，需要在PHP脚本的最前面加上一行特殊的代码，使得它能够被执行，这样系统就能知道用什么样的程序要运行该脚本。为Unix系统增加的第一行代码不会影响该脚本在Windows下的运行，因此可以用该方法编写跨平台的脚本程序。

windows下建立批处理文件，在批处理文件中执行php脚本

```
# 编辑脚本文件，脚本文件的开头#!，用来指定脚本执行时的环境，即使有php解释执行

#!/usr/bin/php
<?php
    var_dump($argv);
?>

# 设置执行权限
$ chmod 755 test

# 执行命令，传递参数-h，--和foo三个参数
$ ./test -h -- foo

# 脚本输出信息：
# array(4) {
#   [0]=>
#   string(6) "./test"
#   [1]=>
#   string(2) "-h"
#   [2]=>
#   string(2) "--"
#   [3]=>
#   string(3) "foo"
# }
```