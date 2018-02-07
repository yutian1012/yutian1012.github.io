---
title: webservice服务使用JDK方式实现
tags: [webservice]
---

使用JDK方式实现webservice服务器端

### 1. 定义提供服务的接口

使用@WebService注解标注接口，使用@WebMethod注解标注接口中定义的所有方法

```
package org.ipph.web.webservice;

import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService
public interface IWebService {
    @WebMethod
    String sayHello(String name);
    @WebMethod
    String save(String name,String pwd);
}
```

注：这里的注解类是JDK提供的

### 2. 接口实现类

使用@WebService注解标注实现类

```
@WebService
public class WebServiceImpl implements IWebService{

    @Override
    public String sayHello(String name) {
        System.out.print("WebService sayHello "+name);
        return "sayHello "+name;
    }

    @Override
    public String save(String name, String pwd) {
        System.out.println("WebService save "+name+"， "+pwd);
        return "save Success";
    }
}
```

### 3. 发布webservice（以应用的方式发布）

使用Endpoint(终端)类发布webservice

```
package org.ipph.web.webservice;

import javax.xml.ws.Endpoint;

public class WebServicePublish {
    public static void main(String[] args) {
        //定义WebService的发布地址，这个地址就是提供给外界访问Webervice的URL地址，URL地址格式为：http://ip:端口号/xxxx
        String address = "http://10.33.6.199:8989/WS_Server/Webservice";
        //使用Endpoint类提供的publish方法发布WebService，发布时要保证使用的端口号没有被其他应用程序占用
        Endpoint.publish(address , new WebServiceImpl());
        System.out.println("发布webservice成功!");
    }
}
```

注：这里我们编写了一个WebServicePublish类来发布WebService，如果是Web项目，那么我们可以使用监听器或者Servlet来发布WebService。

注2：使用EndPoint.publish()方法将会新开启一个线程，所以并不会影响主线程的运行，所以运行主方法时，控制台仍然可以看到有输出：发布webservice成功的信息。

注3：这里的服务ip是本地的ip地址

### 4. 访问服务

访问地址：

```
http://10.33.6.199:8989/WS_Server/Webservice
```

注：但不能使用localhost来访问。

![](/images/java_structure/webservice/webservicepublish.png)

### 5. 在web容器中发布（监听中发布）

可以使用监听器或者Servlet来发布WebService

```
package org.ipph.web.webservice;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
import javax.xml.ws.Endpoint;

@WebListener
public class WebServicePublishListener implements ServletContextListener{

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        String address = "http://10.33.6.199:8989/WS_Server/Webservice";
        Endpoint.publish(address , new WebServiceImpl());
        System.out.println("发布webservice成功!");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
    }
}
```

注：Servlet3.0提供@WebListener注解将一个实现了特定监听器接口的类定义为监听器，这样我们在web应用中使用监听器时，也不再需要在web.xml文件中配置监听器的相关描述信息了。

注2：Servlet3.0规范的出现，让我们开发Servlet、Filter和Listener的程序在web.xml实现零配置

### 6. 使用servlet发布（servlet中发布）

```
package org.ipph.web.webservice;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.xml.ws.Endpoint;

@WebServlet(loadOnStartup=0,value="")
public class WebServicePublishServlet extends HttpServlet{
    private static final long serialVersionUID = 1L;

    @Override
    public void init() throws ServletException {
        String address = "http://10.33.6.199:8989/WS_Server/Webservice";
        Endpoint.publish(address , new WebServiceImpl());
        System.out.println("发布webservice成功!");
    }
}
```

注：如果是普通的java项目，那么可以专门写一个类用于发布WebService，如果是Web项目，那么可以使用ServletContextListener或者Servlet进行发布。

注2：WebServlet注解上需要提供value，否则不能启动servlet