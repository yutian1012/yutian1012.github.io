---
title: 在java原文件编码
tags: [coding]
---

参考：http://blog.csdn.net/fgstudent/article/details/43191247

javac编译文件也需要读取文件，它使用什么编码呢？

### javac在控制台编译java类文件

1）javac编译java源码文件

通常我们手动建立一个java文件Demo.java，并保存。此时Demo.java文件的编码为ANSI，中文操作系统下就是GBK。然后使用javac命令来编译该源文件。

```
# 运行javac编译命令
javac Demo.java
```

Javac也需要读取java文件，那么javac是使用什么编码来解码我们读取的字节呢？

javac采用了操作系统默认的GBK编码解码我们读取的字节，这个编码正好也是Demo.java文件的编码，二者一致，所以不会出现乱码情况。

2）修改java源码保存文件编码

在保存Demo.java文件时，选择UTF-8保存。此时Demo.java文件编码就是UTF-8了。再使用javac来编译，如果Demo.java里含有中文字符，此时控制台会出现警告信息，也出现了乱码。

是因为javac采用了GBK编码解码我们读取的字节。因为我们文件中的字节是UTF-8编码的，所以会出现乱码。

解决办法就是使用javac的encoding参数来制定我们的解码编码。

```
# 编译时指定编译原文件时所用的编码
javac -encoding UTF-8 Demo.java
```

这里我们指定了使用UTF-8来解码读取的字节，由于这个编码和Demo.java文件编码一致，所以不会出现乱码情况了。

### Eclipse中编译java文件。

在使用IDE开发环境时，我们习惯把Eclipse的编码设置成UTF-8。那么每个项目中的java源文件的编码就是UTF-8。这样编译也从没有问题，也没有出现过乱码。正是因为这样才掩盖了使用javac可能出现的乱码。

那么Eclipse是如何正确编译文件编码为UTF-8的java源文件的呢？

唯一的解释就是Eclipse自动识别了我们java源文件的文件编码，然后采取了正确的encoding参数来编译我们的java源文件。功劳都归功于IDE的强大了。

### 使用Ant来编译java文件。

Ant也是我们常用的编译java文件的工具。

首先，必须知道Ant后台其实也是采用javac来编译java源文件的，那么可想而知，Ant中也会存在编译乱码的问题。如果我们使用Ant来编译UTF-8编码的java源文件，并且不指定如何编码，那么也会出现乱码的情况。所以Ant的编译命令javac有一个属性encoding允许我们指定编码，如果我们要编译源文件编码为UTF-8的java文件，那么我们的命令应该如下：

```
<javac destdir="${classes}" target="1.4" source="1.4" 
    deprecation="off" debug="on" debuglevel="lines,vars,source" 
    optimize="off" encoding="UTF-8">
    ...
</javac>
```

指定了编码也就相当于”javac –encoding”了，所以不会出现乱码了。