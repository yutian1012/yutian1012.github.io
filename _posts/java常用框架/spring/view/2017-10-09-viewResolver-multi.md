---
title: spring mvc支持混合视图
tags: [spring]
---

参考：http://blog.csdn.net/z69183787/article/details/40426603

### 方案

1）使用viewName的后缀区分解析器（推荐）

需要自定义视图解析器，继承ViewResolver，通过不同的视图名称后缀选择不同的解析器来处理。

2）使用ResourceBundleViewResolver来处理

ResourceBundleViewResolver可以让视图解释器支持解析多种视图，而使用的UrlBasedViewResolver只支持解释单一类型的视图。

ResourceBundleViewResolver通过读取xxx.properties文件，进行视图解析。文件位于classpath路径下，xxx的值是通过设置ResourceBundleViewResolver的basename属性来设定的。

xxx.properties文件

```
welcome.class=org.springframework.web.servlet.view.JstlView
welcome.url=/WEB-INF/pages/welcome.jsp
```

注：welcome是要匹配的视图名称，class是视图的类型，url属性是视图的url位置。

执行过程：

当控制器返回一个名为welcome的视图时，ResourceBundleViewResolver将在xxx.properties”文件中查找以welcome起始的键，使用指定的类型对应的解析器进行解析，并返回相对应的视图URL：/WEB-INF/pages/welcome.jsp给DispatcherServlet。