---
title: log4j多个web应用的冲突
tags: [log4j]
---

### log4j多个web应用的冲突

参考：http://elf8848.iteye.com/blog/2008595

1）设置webapp的根目录

```
<context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>webapp.root</param-value>
</context-param>
```

注："webapp.root"这个字符串可以随便写任何字符串。如果不配置默认值是"webapp.root"。

2）获取

```
System.getProperty("webapp.root")
```

注：可以用System.getProperty方法来动态获项目的运行路径。一般返回结果例如：/usr/local/tomcat6/webapps/项目名。

如果将webAppRootKey的值变为webapp.root1，则调用getProperty时就会使用webapp.root1来获取。

3）原理分析

Spring通过org.springframework.web.util.WebAppRootListener这个监听器来运行时的项目路径。但是如果在web.xml中已经配置了org.springframework.web.util.Log4jConfigListener这个监听器，则不需要配置WebAppRootListener了。因为Log4jConfigListener已经包含了WebAppRootListener的功能。

### WebAppRootListener监听器

这个监听器会在web上下文初始化的时候，调用webUtil的对应方法，首先获取根传递进来的servletContext得到物理路径，String path=servletContext.getRealPath("/");然后找到context-param的webAooRootKey对应的param-value，把param-value的值作为key，上面配置的是"myroot", 接着执行System.setProperty("myroot",path)。这样在web中就可以使用System.getProperty("myroot")来获取系统的绝对路径。

1）如果只配置了监听器，没有配置webAppRootKey，默认wenAppRootKey对应的param-value的值为webapp.root。

2）上面得到系统路径是Spring的做法，和平时自己采用的方法基本一致，都是写一个监听器，然后得到物理路径后手动放入System中，一般还会在这个监听器中加载配置文件，获取配置文件的值。