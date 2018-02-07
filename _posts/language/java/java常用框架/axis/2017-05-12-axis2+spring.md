---
title: axis2与spring web项目整合
tags: [webservice]
---

参考：http://blog.csdn.net/kris234seth/article/details/50474575

### 导入依赖jar包

```
<!-- axis 1.6.2 -->
    
<dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2</artifactId>
    <version>${axis2.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2-spring</artifactId>
    <version>${axis2.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2-transport-http</artifactId>
    <version>${axis2.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.axis2</groupId>
    <artifactId>axis2-transport-local</artifactId>
    <version>${axis2.version}</version>
</dependency>
```

### 配置web.xml

```
<!--加入Axis2支持  -->  
<servlet>  
       <servlet-name>AxisServlet</servlet-name>  
       <servlet-class>org.apache.axis2.transport.http.AxisServlet</servlet-class>  
       <load-on-startup>1</load-on-startup>  
 </servlet>  
 <servlet-mapping>  
       <servlet-name>AxisServlet</servlet-name>  
       <url-pattern>/services/*</url-pattern>  
 </servlet-mapping> 
```

### 创建webservice类

```
package org.ipph.web.webservice.axis;

import java.util.Random;

public class SpringWebServiceDemo {
    public String springHello(){  
        return "hello spring-axis2";   
    }  
      
    public int getAge(){  
        return new Random().nextInt(80);  
    }  
      
    public void update(){  
        System.out.println("update something..");  
    } 
}
```

### 将类的实例化交由spring来实现

在applicationContext.xml文件中配置

```
<bean id="springWebService" class="org.ipph.web.webservice.axis.SpringWebServiceDemo"></bean>
```

### copy axis2的相关目录

将下载的axis.war包解压缩，拷贝axis2-web，/WEB-INF/conf，/WEB-INF/modules，/WEB-INF/services目录到项目中。

![](/images/java_structure/webservice/axis2/axis2-dir.png)

拷贝后的文件结构

![](/images/java_structure/webservice/axis2/axis2+spring.png)

conf/axis.xml,包括了axis2的常见配置
services用于存放要部署的WebService
axis2-web存放管理webservice的jsp页面

### 新建services.xml文件

在项目的WEB-INF下新建如下层次结构目录 ： services/springServices/META-INF/services.xml。

文件内容

```
<?xml version="1.0" encoding="UTF-8"?>  
<serviceGroup>    
   <service name="springService">    
      <description>Web Service</description>    
      <!--   
           作用类似于普通配置中的 ServiceClass ，都是用来创建服务类对象，  
           只不过普通配置使用反射来创建  , 加入Spring之后，对象的创建交给了Spring的IOC容器  
      -->  
      <parameter name="SpringBeanName">springWebService</parameter>  
      <parameter name="ServiceObjectSupplier">org.apache.axis2.extensions.spring.receivers.SpringServletContextObjectSupplier</parameter>    
       <!--   
                   在这里最值得注意的是<messageReceivers>元素，该元素用于设置处理WebService方法的处理器。  
                   例如，getAge方法有一个返回值，因此，需要使用可处理输入输出的RPCMessageReceiver类，  
                   而update方法没有返回值，因此，需要使用只能处理输入的RPCInOnlyMessageReceiver类。  
        -->  
      <messageReceivers>    
             <messageReceiver mep="http://www.w3.org/2004/08/wsdl/in-out" class="org.apache.axis2.rpc.receivers.RPCMessageReceiver" />    
             <messageReceiver mep="http://www.w3.org/2004/08/wsdl/in-only" class="org.apache.axis2.rpc.receivers.RPCInOnlyMessageReceiver" />    
      </messageReceivers>    
   </service>    
</serviceGroup>
```

### 部署并启动服务

访问地址

```
http://localhost:8080/springTest/services/listServices
```

可以看到发布的web service.

```
http://localhost:8080/springTest/services/springService?wsdl
```

### 删除项目目录测试

删除axis2-web，/WEB-INF/conf，并清空/WEB-INF/modules内容，再次运行，wsdl路径依然可以访问。

```
http://localhost:8080/springTest/services/springService?wsdl
```

![](/images/java_structure/webservice/axis2/axis2+spring-withoutdir.png)

注：axis2-web是为了方便管理web service的发布信息而存在的。

因为删除了axis-web目录，访问下面链接失败

```
http://localhost:8080/springTest/services/listServices
```

错误信息

```
javax.servlet.ServletException: File &quot;/axis2-web/listServices.jsp&quot; not found
```