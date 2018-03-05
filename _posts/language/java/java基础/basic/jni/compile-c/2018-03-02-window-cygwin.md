---
title: windows下cygwin编译C文件
tags: [jni]
---

*使用Cygwin开发jni是为了在linux平台上运行相关代码*

Cygwin是一个在windows平台上运行的类UNIX模拟环境。cygwin是一个windows平台上的unix模拟环境，主要是通过重新编译，将posix系统上的软件移植到windows上。

注：可以利用Cygwin中提供的g++对c文件进行编译。

无法在windows平台上直接运行java程序访问c代码，因为c代码的执行必须运行在cygwin环境下。除非在cygwin中运行java，然后在同一个环境中执行c程序。

参考：http://blog.csdn.net/tps2046/article/details/38947031

1）安装Cygwin

安装完成后，会在桌面上提供Cygwin64 Terminal应用窗口。打开Cygwin64 Terminal窗口工具，使用时类似于linux系统。挂接的跟路径为软件的安装目录。

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

2）安装gcc

再次运行cygwin的安装文件setup.exe，一直下一步到Select Packages，然后Search gcc，把把需要的包Default-->Install即可。

![](/images/middleware/jni/cygwin/installgcc.png)

注：安装目录要与之前安装的目录匹配，这里不需要再次手动设置，安装程序已经记住了软件安装位置，因此不需要修改，直接next到select package界面选择安装gcc。

查看gcc是否正确安装

```
gcc -v
```

3）编写c程序

在E:\work\test\c目录下创建test.c文件

```
#include <stdio.h>

int main(){
    printf("Helloworld");
    return 0;
}
```

4）编译并运行test.c文件

```
# cygwin中切换目录到E:\work\test\c下
cd E:/work/test/c
# 执行编译命令
gcc -o test test.c
# 运行生成的test.exe
./test.exe
```

5）cygwin中运行java

首先在windows本地安装jdk，然后在Path环境变量中配置jdk的bin目录。在cygwin中会自动读取windows系统设置的环境变量。因此cygwin中是可以访问到java命令的。

注：cygwin会使用宿主机的java来运行命令。

```
# 在cygwin中运行java -version，查看是否可以正确运行java
java -version

# 查看cygwin的环境变量
# 可以看到jdk的路径信息：/cygdrive/e/work/installer/jdk1.8.0_131/bin
echo $PATH
```

注：环境配置完成后就可以编写java调用c的代码来测试了。