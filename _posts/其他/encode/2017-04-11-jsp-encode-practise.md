---
title: jsp页面中涉及到的编码实例
tags: [other]
---

### 1. jsp中不指定编码

1）出现场景：

项目中使用freemarker来处理页面，因此没有在jsp页面中添加page指令。即没有下面的设置。

```
<%@page contentType="text/html; charset=UTF-8" %>
```

注：因为freemarker解析时不会处理jsp中的语法，只会原样输出。该页面出现在index.jsp中，恰好没被freemarker引擎解析，而最终被jsp解析了。

2）问题的原因

在jsp页面中不设置page指令指定编码，那么servlet（jsp本质上就是servlet）在输出时就无法知道页面是以什么方式被编码出去，因此会使用iso-8859-1的编码进行页面输出编码，浏览器解析就会出现错误。

响应头信息

```
Content-Type:"text/html;charset=ISO-8859-1"
```

3）修改

方式一：在jsp页面的head中添加meta

```
<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
```

结果：不起作用

方式二：CharacterEncodingFilter过滤器

```
<filter>  
    <filter-name>characterEncodingFilter</filter-name>  
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
    <init-param>  
        <param-name>encoding</param-name>  
        <param-value>UTF-8</param-value>  
    </init-param>  
    <init-param>  
        <param-name>forceEncoding</param-name>  
        <param-value>true</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>characterEncodingFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

注：主要对post提交的参数进行编码过滤

结果：不起作用，但响应头发生了变化，变成了UTF-8。依然乱码说明服务器端编码输出时使用的不是UTF-8，因此，即使浏览器使用UTF-8解析页面也无法正确解码中文。

```
Content-Type:"text/html;charset=UTF-8"
```

方式三：

对于jsp解析的页面一定要添加page指令，并在page指令中指定编码。

```
<%@page contentType="text/html; charset=UTF-8" %>
```

注：freemarker解析页面的编码是在其引擎中配置的，而一旦交由jsp解析，这需要告知jsp编码输出的编码方式。
