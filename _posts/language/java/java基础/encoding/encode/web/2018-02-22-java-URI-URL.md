---
title: URI和URL的区别
tags: [coding]
---

1）概念问题：

URI（uniform resource identifier）统一资源标识符，用来唯一的标识一个资源。

URL（uniform resource locator）统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。

注：URI是以一种抽象的，高层次概念定义统一资源标识，而URL则是具体的资源标识的方式。

2）服务器端代码区别

访问地址：
```
http://localhost:8080/java_web/UrlEncodeServlet?hello=你好
```

代码执行：

```
System.out.println(request.getRequestURI());
System.out.println(request.getRequestURL());

# 输出
/java_web/UrlEncodeServlet
http://localhost:8080/java_web/UrlEncodeServlet
```

注：URI实例可以代表绝对的，也可以是相对的；而URL类则不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的。

参考：http://www.ruanyifeng.com/blog/2010/02/url_encoding.html