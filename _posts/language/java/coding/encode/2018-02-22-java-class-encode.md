---
title: 在java中class文件的编码及println输出编码
tags: [coding]
---

java最终的class文件都是以unicode编码的。

Jvm在运行时其内部都是采用unicode编码的，其实在输出时，又会做一次编码的转换。

### java中采用Sysout.out.println输出。

1）java代码

```
Sysout.out.println("我们");
```

经过正确的解码后"我们"是unicode保存在内存中的，但是在向标准输出（控制台）输出时，jvm又做了一次转码，它会采用操作系统默认编码（中文操作系统是GBK），将内存中的unicode编码转换为GBK编码，然后输出到控制台。

因为我们操作系统是中文系统，所以往终端显示设备上打印字符时使用的也是GBK编码。因为终端的编码无法手动改变，所以这个过程对我们来说是透明的，只要编译时能正确转码，最终的输出都将是正确的，不会出现乱码。

2）Eclipse控制台设置输出编码：

在Eclipse中可以设置控制台的字符编码，具体位置在Run Configuration对话框的Common标签里，我们可以试着设置为UTF-8，此时的输出就是乱码了。因为输出时是采用GBK编码的，而显示却是使用UTF-8，编码不同，所以出现乱码。

### jsp中使用out.println()输出到客户端浏览器

1）jsp页面输出

Jsp编译成class后，如果输出到客户端，也有个转码的过程。Java会采用操作系统默认的编码来转码，那么tomcat采用什么编码来转码呢？

tomcat是根据contentType的charset参数来转码的，contentType用来设置tomcat往浏览器发送HTML内容所使用的编码。

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
```

Tomcat根据这个编码来转码内存中的unicode。经过转码后tomcat输出到客户端的字符编码就是utf-8了。

2）浏览器获取显示编码

浏览器怎么知道采取什么编码格式来显示接收到的内容呢？

这就是contentType的charset属性的第三个作用了，这个编码会在HTTP响应头中指定以通知浏览器。浏览器使用http响应头的contentType的charset属性来显示接收到的内容。

总结一下contentType charset的三个作用：

1）在没有pageEncoding属性时，tomcat使用它来解码读取的jsp文件。

2）tomcat向客户端输出时，使用它来编码发送的内容。

3）通知浏览器，应该以什么编码来显示接收到的内容。

### 实例

新建一个index.jsp文件，该文件编码为GBK,在jsp开头我们写上如下代码：

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="GBK"%>
```

这里的charset和pageEncoding不同，但是也不会出现乱码。

首先tomcat读取jsp内容，并根据pageEncoding指定的GBK编码，将读取的GBK字节解码并转换为unicode字节码保存在class文件中。然后tomcat在输出时，out.println()使用charset属性将内存中的unicode转换为utf-8编码，并在响应头中通知浏览器，浏览器以utf-8显示接收到的内容。整个过程没有一次转码错误，所以就不会出现乱码情况。