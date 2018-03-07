---
title: windows下使用MinGW-w64编译C文件
tags: [jni]
---

*MinGW编译的软件是为了在windows上运行，与cygwin是相反的*

MinGW提供了C语言的编译运行工具，开发的能够保证在windows下正常运行。

1）区分MinGW和MinGW-w64

MinGW和MinGW-w64是两个不同的项目，MinGW本身已经很久没有更新了，故不推荐。

MinGW，全称Minimalist GNU for Windows, 是GCC编译器和GNU Binutils在Windows平台的移植版本。MinGW-w64原是其分支，后来成为独立发展的项目。由于仅有MinGW-w64被GCC官方所支持, 而MinGW早已停止更新, 因此推荐使用MinGW-w64

2）MinGW

下载软件：mingw-get-setup.exe，是图形化的安装助手。

注：安装过程很可能会失败，主要是网络问题。

3）MinGW-w64

下载离线安装包：https://sourceforge.net/projects/mingw-w64/files/?source=navbar

![](/images/middleware/jni/mingw/download.png)

针对自身的系统选择安装相应的版本

a.architecture，64位系统需要选择x86_64，

b.threads，如果没有跨平台编译需求，就选win32。如果有的话选posix

c.exceptions，dwarf、sjlj的异常模型选择，推荐使用dwarf即所谓dw2，这个模型便于调试。

注：解压后将bin设置到环境变量中

4）运行命令查看GCC版本

```
# 在cmd中运行命令查看GCC版本
gcc -v
```

### 编译运行C

1）新建test.c文件

```
#include <stdio.h>

int main(){
    printf("Helloworld");
    return 0;
}
```

2）使用gcc编译文件

```
# 使用gcc编译命令，生成test.exe命令
gcc test.c -o test
```

### 编译c++文件

1）新建test.cpp文件

```
#include <stdio.h>

int main(){
    printf("Helloworld");
    return 0;
}
```

2）使用g++编译c++文件

```
# 使用g++编译命令，生成testcpp.exe命令
g++ test.cpp -o testcpp
```

3）修改test.cpp文件

使用c++特有的语法编辑文档

```
#include <iostream>

int main(){
    std::cout << "HelloWorld!" << std::endl;
    return 0;
}
```