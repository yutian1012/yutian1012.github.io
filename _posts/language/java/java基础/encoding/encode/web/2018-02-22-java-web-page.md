---
title: JSP页面与服务器端中文乱码问题
tags: [coding]
---

1）jsp页面设置编码

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
```

保证页面中的中文都是有UTF-8进行编码（包括js代码中的中文）。

注：针对tomcat和浏览器都是有效的，即tomcat会通过该编码值来解析jsp源文件，浏览器会通过该获取页面显示的编码。

注：该jsp命令会向响应头中设置Content-Type:text/html;charset=UTF-8

2）ajax方式提交的数据

a.get方式提交的数据

js中的encodeURI和encodeURIComponent能对中文等字符进行UTF-8编码。

b.post方式提交的数据

不需要进行特殊的处理，并且会在请求头中设置Content-Type编码为UTF-8

3）服务器端servlet

a.从request获取请求提供的数据（指针对post方式提交的数据）

```
request.setCharacterEncoding("UTF-8");
```

注：URI中的编码不能通过上述方式解码。

b.向response中设置输出编码，只会将输出的数据使用UTF-8进行编码

```
response.setCharacterEncoding("UTF-8")
```

c.告知浏览器返回数据的编码

```
response.setContentType("text/html;charset=UTF-8");
```

4）设置URI编码

在tomcat的server.xml中设置URI编码

```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" 
    redirectPort="8443" URIEncoding="UTF-8"
/>
```