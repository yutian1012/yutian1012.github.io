---
title: 浏览器页面编码
tags: [coding]
---

1）JSP页面设置编码

```
<%@page contentType="text/html; charset=UTF-8" %>
```

2）响应头中会返回对应的文档编码信息，浏览器就能够进行解析页面。

```
Content-type:text/html;charset=UTF-8
```

3）页面的中文连接

```
<a href="http://localhost:8080/java_web/UrlEncodeServlet?hello=你好">中文参数</a>
```

Content-type中的UTF-8编码会在浏览器接收到响应内容时，使用此编码进行页面的解析渲染，因此页面中的中文连接也会使用UTF-8进行编码，传递到后台自然是使用UTF-8进行编码的值（解析网页的编码）。

### servlet设置浏览器解析返回数据的编码

在servlet设置浏览器解析编码

```
response.setContentType("text/html;charset=UTF-8");
```

注：这种方式也会在响应头中添加Content-type，且编码为UTF-8，告知浏览器使用什么编码来解析返回的文档数据。