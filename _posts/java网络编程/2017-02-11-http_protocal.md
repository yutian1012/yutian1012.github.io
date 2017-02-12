---
title: http协议
tags: [web]
---

HTTP协议（HyperText Transfer Protocol，超文本传输协议）是用于从WWW服务器传输超文本(html是超文本标记语言)到本地浏览器的传输协议。

在浏览器的地址栏里输入的网站地址叫做URL (Uniform Resource Locator，统一资源定位符)。

### 1. Http是一个基于请求/响应模式的，无状态的协议

注：可以通过某种手段，使无状态变成有状态

### 2. http1.0协议
1）通信协议传输

![](/images/java_net/http/http1.0.png)

2）Http1.0的特点：

在整个通信过程中需要不断的建立连接，一次请求，一次连接，请求完毕后立即释放连接。不支持连续的请求。现在的Web页面中不仅有HTML文本，还有图片，视频等资源。
例如：包含100张图片的网页，要建立101次连接。因为，一个包含有许多图像的网页文件中并没有包含正真的图像数据内容，而只是指明了这些图像的URL地址，当web浏览器访问这个网页文件时，浏览器首先要发出针对该网页文档中的HTML内容时，发现其中的img图像标签后，浏览器将根据img标签中的src属性所指定的URL地址再次向服务器发出下载图像数据的请求。

注：最主要的特点就是不支持持续连接

### 3. http1.1协议
1）通信协议传输（在同一个连接上可发出多个请求）

![](/images/java_net/http/http1.1.png)

2）特点：

在Http1.1版本中，给出了一个持续连接（Persistent Connections）机制。浏览器可以通过建立这种连接之后，发送请求并得到响应，然后继续发送请求并再次得到回应。而且，客户端还可以发送流水线请求，也就是说，客户端可以连续发送多个请求，而不用等待每一个相应的到来。

### 4. http请求信息
Http请求信息包括：请求行，报头信息（请求头），请求正文

1）请求行：
请求行以一个方法符号开头，后面跟着请求URI和协议的版本，以CRLF（回车换行）为结尾。请求行中以空格分隔各个字段信息。如：GET /sample.jsp HTTP/1.1 (CRLF).
请求常用方法：GET，POST，HEAD，DELETE，TRACE，CONNECT，OPTIONS，PUT。

![](/images/java_net/http/httpmethod.png)

注：当我们通过在浏览器的地址栏中直接输入网址的方式去访问网页的时候，浏览器采用的就是GET方法向服务器获取资源。当使用表单时，可以通过GET和POST方法来进行传递，通过相应的属性进行设置（必须大写）。

2）报头信息（消息报头/请求头）
请求头包含许多有关的客户端环境和请求正文的有用信息。例如，请求头可以声明浏览器所用的语言，请求正文的长度等。

如：Accept:/images/java_net/gif./images/java_net/jpeg.*/*，Accept-Encoding:gzip,deflate等

3）请求正文

请求头和请求正文之间是一个空行，这个行非常重要，它表示请求头已经结束，接下来的是请求正文。

请求正文中可以包含客户提交的查询字符串信息：username=jinqiao&password=1234

### 5. http相应信息
Http响应也是由三个部分组成，分别是：状态行，消息报头，响应正文。

1）状态行

状态行由协议版本，数字形式的状态代码，相应的状态描述组成，各个元素之间以空格分隔，结尾以CRLF（回车换行）结束。如：HTTP/1.1 200 OK (CRLF)

http状态代码与状态描述：

1xx：指示信息--表示请求已接收，继续处理

2xx：成功--表示请求已经被成功接收，理解，接受

3xx：重定向--要完成请求必须进行更进一步的操作

4xx：客户端错误--请求有语法错误或请求无法实现

5xx：服务器端错误--服务器未能实现合法的请求

常见的状态代码：

![](/images/java_net/http/httpcode.png)

2）消息报头（相应头）

响应头(Response Header)响应头也和请求头一样包含许多有用的信息，例如服务器类型、日期时间、内容类型和长度等：Server:Apache Tomcat/5.0.12，Last-Moified:Mon,6 Oct 2003 13:23:42 GMT。

3）响应正文

响应正文就是服务器返回的HTML页面。

### 6. 实验
1）使用telnet工具

win7下打开telnet服务，控制面板--程序和功能--点击打开或关闭Windows功能--在对话框中勾选telnet客户端。

2）Telnet是进行远程登录的标准协议和主要方式它为用户提供了在本地计算机上完成远程主机工作的能力。

3）运行

第一步：打开cmd窗口，输入telnet www.baidu.com 80，回车，会启动telnet命令行窗口

注：使用的协议是telnet而不是Http协议，因此不能使用http://www.baidu.com，而直接使用域名。访问时指定端口号80。

第二步：输入请求：ctrl+']'，可以打开本地回显功能

第三步：进入telent的编辑视图：直接按下回车

第四步：编辑发生命令，将下面的命令直接粘贴到telent的编辑框中，两次回车

```
GET / HTTP/1.1
Host:www.baidu.com
```

注：字段之间使用空格分隔，需要两次回车，第一次回车表示发送的数据准备好了。

![](/images/java_net/http/httptelnet.png)

第五步：查看HEAD方法

```
HEAD / HTTP/1.1
Host:www.baidu.com
```

![](/images/java_net/http/httptelnethead.png)

### 7. telnet连接tomcat服务器查看
1）运行启动tomcat

2）telnet localhost 8080

3）在telnet编辑框中输入命令，查看

```
GET / HTTP/1.1
Host:localhost
Connection:Keep-Alive
```

![](/images/java_net/http/httptelnettomcat.png)
