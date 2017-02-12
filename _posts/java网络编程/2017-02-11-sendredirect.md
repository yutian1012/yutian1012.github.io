---
title: 请求转发与重定向
tags: [web]
---

请求转发与重定向的比较

### 1. 比较

```
<a href="jspForward2.jsp">jspFoward2.jsp</a>
<jsp:forward page="jspForward2.jsp">
```

1）代码1：点击后发送一个新的请求

2）代码2：属于转发请求，是同一个请求。

### 2. 问题
jspForward1.jsp要把请求转发给jspForward2.jsp，在转发的时候，希望把用户名“小新”传给jspForward2.jsp，如何实现？

方式一：

```
request.setAttribute("username","小新");
```

方式二：

```
<jsp:forward page="jspForward2.jsp?username=小新" />
```

注：方式二可能会有乱码问题

方式三：

```
request.getResquestDispathcer("/jspForward2.jsp?username=小新").forward(request,response);
```

注：转发的过程都是在服务器端完成的，跟客户端没有任何关系，让两个页面共享同一个request对象。

### 3. 重定向

```
response.sendRedirect("test4.jsp");
```

### 4. 区别
在使用请求转发过程中可以向同一个request中放置内容，在转发的页面可以从该request中读取信息，使用的是同一request对象。--整个过程都方式在服务器端

重定向，是属于两次请求。

### 5. 问题：
表单重复提交的情况：填写完成后，点击提交后刷新，使用请求转发就会造成订单的重复提交。

解决方法：使用重定向方式解决，提交表单后，处理完成请求，重定向到表单提交页面并提示表单成功提交信息。