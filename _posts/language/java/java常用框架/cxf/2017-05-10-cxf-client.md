---
title: CXF客户端调用
tags: [webservice]
---

根据第三方提供的wsdl服务地址，实现webservice服务调用。这部分代码是需要配置在客户端的，及实现客户端访问某台服务器提供的webservice服务。

1）引入jar包依赖

```
<!-- cxf 3.1.11版本-->
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-core</artifactId>
    <version>${cxf.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxws</artifactId>
    <version>${cxf.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http</artifactId>
    <version>${cxf.version}</version>
</dependency>
```

2）JaxWsDynamicClientFactory方式（方式一）

使用JaxWsDynamicClientFactory类动态创建客户端访问对象Client。通过传递方式方法名和相关参数实现webservice调用。

```
public class CxfClient {
    public static void main(String[] args) {
        JaxWsDynamicClientFactory clientFactory=JaxWsDynamicClientFactory.newInstance();
        Client client = clientFactory.createClient("http://10.33.6.199:8989/WS_Server/Webservice?wsdl");  
        Object[] result;
        try {
            result = client.invoke("sayHello", "KEVIN");
            System.out.println(result[0]);
        } catch (Exception e) {
            e.printStackTrace();
        }  
    }
}
```

注：DynamicClientFactory也可以实现动态创建客户端

```
DynamicClientFactory dcf = DynamicClientFactory.newInstance(); 
```

3）使用spring集合cxf的xml方式配置bean

a.创建wbservice接口类

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

b.配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  
  xmlns:jaxws="http://cxf.apache.org/jaxws"
  
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://cxf.apache.org/jaxws http://cxf.apache.org/jaxws">

    <jaxws:client id="webService" 
        serviceClass="org.ipph.web.webservice.IWebService" 
        address="http://localhost:7777/hello" />
</bean>
```

4）测试类

创建一个servlet，通过WebApplicationContextUtils获取spring容器上下文，并从容器中获取相关的webservice客户端访问对象，从而实现webservice调用。

```
@WebServlet(loadOnStartup=1,urlPatterns="/cxfClient")
public class CxfServletClient extends HttpServlet{
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        //调用ws
        ApplicationContext context= WebApplicationContextUtils
            .getWebApplicationContext(req.getServletContext());
        IWebService service=(IWebService) context.getBean("webService");
        service.sayHello("ssss");
    }
}
```

5）代码方式实现

```
import com.cc.entity.User;
import com.cc.service.UserService;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.springframework.context.support.ClassPathXmlApplicationContext;


public class Main {

    public static void main(String[] args) {
        //使用JaxWsProxyFactoryBean调用soap服务，但是这种方式需要需要生成接口文件
        System.out.println("***********JaxWsProxyFactoryBean***********");
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();
        factory.setServiceClass(UserService.class);
        factory.setAddress("http://localhost:9000/serviceTest/service/userService");
        UserService userServiceJaxBean = (UserService)factory.create();
        User userJaxBean = userServiceJaxBean.getName("zhangsan");
        System.out.println(userJaxBean.getTelephone());

        //测试jaxws:client，soap服务client调用
        System.out.println("******************soap jaxws:client**************");
        ClassPathXmlApplicationContext cxt = new ClassPathXmlApplicationContext(
                new String[]{"client.xml"});
        UserService us = (UserService)cxt.getBean("client");
        User user = us.getName("aaa");
        System.out.println(user.getTelephone());

    }
}
```