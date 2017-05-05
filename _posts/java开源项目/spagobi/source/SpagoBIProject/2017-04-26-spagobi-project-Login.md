---
title: 开源spagobi--系统登录
tags: [spagobi]
---

### 1. 登录链接

```
http://localhost:8080/SpagoBI/servlet/AdapterHTTP?PAGE=LoginPage&NEW_SESSION=TRUE

//请求主体
isInternalSecurity=true&userID=biadmin&password=biadmin
```

涉及到的类包括：EncodingFilter，SpagoBICoreCheckSessionFilter（判断session是否超期），ProfileFilter，AntiInjectionFilter（web中配置的过滤器），AdapterHttp（servlet实现类，用于适配http请求--可视为前端处理器DispatchServlet），DefaultPage（Page模块的处理类--可视为controller）和LoginModule（登录模块处理类--可视为service）。

![](/images/open/spagobi/source/login-class.png)

### 2. 请求参数的封装




登录参数放置到了serviceRequest（SourceBean对象）中。