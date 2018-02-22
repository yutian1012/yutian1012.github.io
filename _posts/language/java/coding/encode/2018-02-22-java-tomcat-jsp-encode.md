---
title: tomcat中编译jsp的情况
tags: [coding]
---

tomcat编译jsp时也要读取文件，此时tomcat采用什么编码来读取文件？

1）设置pageEncoding属性

我们通常会在jsp开头写上如下代码：

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
```

pageEncoding这个属性就是告诉tomcat采用什么编码来读取jsp文件的。它应该和jsp文件本身的编码一致。

如新建个jsp文件，设置文件编码为GBK,那么此时我们的pageEncoding应该设置为GBK，因为我们写入文件的字符就是GBK编码的，tomcat读取文件时采用也是GBK编码，所以能保证正确的解码读取的字节。不会出现乱码。

如果把pageEncoding设置为UTF-8，那么读取jsp文件过程中转码就出现了乱码。

2）contentType的charset属性

我们常常不写pageEncoding这个属性，但是也没出现过乱码，这是怎么回事呢？

那是因为如果没有pageEncoding属性，tomcat会采用contentType中charset编码来读取jsp文件，jsp文件编码通常设置为UTF-8，contentType的charset也设置为UTF-8，这样tomcat使用UTF-8编码来解码读取的jsp文件，二者编码一致也不会出现乱码。

3）两个属性都不设置，使用ISO-8859-1编码来编译jsp文件

如果我既不设置pageEncoding属性，也不设置contentType的charset属性，那么tomcat会采取什么编码来解码读取的jsp文件呢？

答案是iso-8859-1，这是tomcat读取文件采用的默认编码，如果用这种编码来读取文件显然会出现乱码。

### 补充

jsp中contentType的charset属性的三个作用

1）在没有pageEncoding属性时，tomcat使用它来解码读取的jsp文件。

2）tomcat向客户端输出时，使用它来编码发送的内容

```
out.println()
```

3）通知浏览器，应该以什么编码来显示接收到的内容