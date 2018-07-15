---
title: web服务器比较
tags: [web]
---

参考：http://blog.csdn.net/zhgn2/article/details/14774603

### Apache

Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。

### tomcat

Tomcat 服务器是一个免费的开放源代码的Web应用服务器（主要用于解析servlet/JSP,同时具备http服务）。

注：使用java的web服务时才会选择。

### jetty

Jetty采用业界最优的开源Java Web引擎，将tomcat内核作为servlet容器引擎。不仅支持 JSP等Java技术，同时还支持其他Web技术的集成，譬如PHP、.NET两大阵营。

### Apache与tomcat

1）Apache支持静态页，Tomcat支持动态的，比如Servlet等。

2）一般使用Apache+Tomcat的话，Apache只是作为一个转发，对JSP的处理是由Tomcat来处理的。

3）Apache可以支持PHP等语言，但要支持java时，就需要结合tomcat来实现了。即在Apache环境下运行JSP的话需要一个解释器来执行JSP网页，而这个JSP解释器就是Tomcat。

4）Apache是Web服务器，Tomcat是应用（Java）服务器，它只是一个Servlet（JSP也翻译成Servlet）容器，可以认为是Apache的扩展，但是可以独立于Apache运行。

5）Apache是专门用了提供HTTP服务的，以及相关配置的（例如虚拟主机、URL转发等等）。Tomcat是Apache组织在符合Java EE的JSP、Servlet标准下开发的一个JSP服务器。

### tomcat与jetty

Tomcat开发的代码只要不使用JSP/Servlet最新规范中的内容，移植到Jetty上不费吹灰之力。

### tomcat与jboss

tomcat是JSP/Servlet容器，jboss是JEE容器，JEE包括JSP/Servlet，JMS，EJB，JAX-WS，JAX-RS，CDI等等。JBoss中的Servlet容器还是Tomcat。与JBoss类似的J2EE应用服务器有：Glassfish（开源）, Geronimo（开源）, WebLogic（商业）, WebSphere（商业）。

### lighttpd

Lighttpd是一个具有非常低的内存开销，cpu占用率低，效能好，以及丰富的模块等特点的web服务器。支持FastCGI，CGI，Auth，输出压缩（outputcompress），URL重写，Alias等重要功能。

### nginx

Nginx是俄罗斯人编写的十分轻量级的HTTP服务器，是一个高性能的HTTP和反向代理服务器，同时也是一个IMAP/POP3/SMTP代理服务器。Nginx并不支持cgi方式运行，原因是可以减少因此带来的一些程序上的漏洞。所以必须使用FastCGI方式来执行PHP程序。

注：Nginx是目前性能最高的HTTP服务器。

### apache与nginx

最核心的区别在于apache是同步多进程模型，一个连接对应一个进程；nginx是异步的，多个连接（万级别）可以对应一个进程。

在性能方面nginx要高于apache，但apache的rewrite比nginx的rewrite强大；模块超多，基本想到的都可以找到。少bug，nginx的bug相对较多。

注：一般来说，需要性能的web服务，用nginx。如果不需要性能只求稳定，那就apache。nginx处理动态请求是鸡肋，一般动态请求要apache去做，nginx只适合静态和反向。

![](/images/middleware/tomcat/web-server.png)