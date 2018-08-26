---
title: cygwin安装
tags: [cygwin]
---

参考：https://blog.csdn.net/chunleixiahe/article/details/55666792

1）下载

https://cygwin.com/install.html

运行程序：setup-x86_64.exe，配置下载信息（install from internet），avaliable download site可以选择（http://mirrors/163.com）

安装相关的包，选择待安装的依赖包信息进行安装，安装完成后，运行terminal窗口会在该依赖环境下运行。如果不知道

![](/images/tools/cygwin/cygwin-setup-package.png)

注：安装完成后把setup-x86_64.exe拷贝到安装目录下，以后可以用来继续添加包

2）启动终端

安装完成后，会在桌面上提供Cygwin64 Terminal应用窗口。打开Cygwin64-Terminal窗口工具，使用时类似于linux系统。挂接的跟路径为软件的安装目录。

a.cygwin的根目录

```
# 打开终端，查看当前目录pwd
pwd
# 进入根目录，并查看根目录下的内容，
# 通过比较根目录下的内容，发现根目录即为cygwin的安装目录
cd / & ls
```

![](/images/middleware/jni/cygwin/homedir.png)

注：cygwin的根目录就是软件安装目录

b.访问系统的c盘

```
# 进入系统c盘
cd c:
# 进入系统c盘
cd /cygdrive/c
```