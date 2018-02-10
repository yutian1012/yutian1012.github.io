---
title: windows安装php环境
tags: [php]
---

参考：http://blog.csdn.net/wlmnzf/article/details/50229407

1）下载php

官网地址：http://windows.php.net

a.vc编译器版本选择

下载php压缩包时，会看到vc字样的信息。这里的VC是指Visual Studio 6编译的。而VC9是用visual studio 2008编译的。VC11，VC14或VC15(分别为Visual Studio 2012，2015或2017编译器)构建的，并且包括性能和稳定性的改进。

b.线程安全的选择

如果选择线程不安全的版本，多个程序同时对一个资源进行写入，就会产生数据错误。

注：最终选择在vc14线程安全的php-7.0.27版本。

2）安装php

安装过程是将下载的zip压缩包解压即可。

3）查看php的安装版本信息

```
# 切换到php的解压目录（安装目录）
cd E:\work\installer\php-7.0.27
# 查看版本号
php -v
```