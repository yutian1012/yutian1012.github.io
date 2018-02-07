---
title: ServletContext
tags: [web]
---

ServletContext,是一个全局的储存信息的空间，服务器开始，其就存在，服务器关闭，其才释放。request，一个用户可有多个；session，一个用户一个；而servletContext，所有用户共用一个。

### 1. ServletContext应用程序的上下文
一个ServletContext对象表示了一个Web应用程序的上下文

1）运行在Java虚拟机中的每一个Web应用程序都有一个与之相关的Servlet上下文。ServletContext对象是Web服务器中的一个已知路径的根，Servlet上下文被定位于http://localhost:8080/项目名。

2）以 /项目名 请求路径（称为上下文路径）开始的所有请求被发送到与此ServletContext关联的Web应用程序。

3）ServletContext,是一个全局的储存信息的空间，服务器开始，其就存在，服务器关闭，其才释放。request，一个用户可有多个；session，一个用户一个；而servletContext，所有用户共用一个。所以，为了节省空间，提高效率，ServletContext中，要放必须的、重要的、所有用户需要共享的线程又是安全的一些信息。

4）Servlet上下文：
Servlet上下文提供对应用程序中所有Servlet所共有的各种资源和功能的访问。

### 2. 初始化参数
ServletContext接口的初始化参数允许servlet访问与web应用相关的上下文初始化参数，这些由应用开发人员在部署描述符中指定：getInitParameter，getInitParameterNames。应用开发人员利用初始化参数传送配置信息。典型的例子是web管理员的e-mail地址或者一个持有关键数据的系统名称。

### 3. 上下文属性
servlet可以通过名称将对象属性绑定到上下文。任何绑定到上下文的属性可以被同一个web应用的其他servlet使用。

ServletContext接口的下列方法允许访问这种功能：setAttribute，getAttribute，getAttributeNames，removeAttribute。

### 4. 资源
ServletContext接口通过下列方法提供对web应用组成的静态内容文档层级的直接访问，包括HTML，GIF和JPEG文件：getResource，getResourceAsStream。

注1：getResource和getResourceAsStream方法以"/"开头的字符串为参数，它指定上下文根路径的资源相对路径。

注2：这些方法不用来获取动态内容。比如，在一个支持JSP规范1的容器中，getResource("/index.jsp")这种形式的方法调用将返回JSP源代码，而不是处理后的输出。

### 5. 临时目录
每一个servlet上下文都需要一个临时存储目录。Servlet容器必须为每一个servlet上下文提供一个私有的临时目录，并且使它可以通过javax.servlet.context.tempdir上下文属性可用。这些属性关联的对象必须是java.io.File类型。但是需要确保一个servlet上下文的临时目录的内容对于该servlet容器上运行的其他web应用的servlet上下文不可见。

### 6. 获取ServletContext的方法
1）在javax.servlet.Filter中直接获取：

```
ServletContext context = config.getServletContext(); 
```

2）在HttpServlet中直接获取：

```
this.getServletContext() 
```

3）在其他方法中，通过HttpRequest获得：

```
request.getSession().getServletContext(); 
```

4）jsp中隐含对象application，就是ServletContext对象。

### 7. 在ServletContext中可以存放共享数据，
ServletContext提供了读取或设置共享数据的方法：setAttribute(String name,Object object),getAttribute(String name)。

参考：http://blog.csdn.net/bryanliu1982/article/details/5214899
