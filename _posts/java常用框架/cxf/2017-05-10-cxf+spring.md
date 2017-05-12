---
title: CXF与spring的整合
tags: [webservice]
---

### 在spring项目中添加cxf支持

1）pom.xml文件

```
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

2）web.xml文件

```
<servlet>
    <servlet-name>cxf</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <!-- 项目启动的时候加载 -->
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>cxf</servlet-name>
    <url-pattern>/cxf/*</url-pattern>
</servlet-mapping>
```


### jaxws:server方式发布

applicationContext.xml文件

```
<!-- webservice注解类,可以是service层类 -->
<bean id="webServiceImpl" class="org.ipph.web.webservice.WebServiceImpl" />

<!-- 让CXF 与Spring整合,通过Spring发布 WS服务 -->
<!-- JAXWS 来发布WS服务 -->
<!-- address:就是此服务的访问地址:  http://localhost:8080/ws/cxf/hello  -->
<jaxws:server address="/hello">
    <jaxws:serviceBean>
        <!-- 此类可以发布为ws服务类 -->
        <ref bean="webServiceImpl" />
    </jaxws:serviceBean>
    <jaxws:inInterceptors>
        <bean class="org.apache.cxf.interceptor.LoggingInInterceptor" />
    </jaxws:inInterceptors>
    <jaxws:outInterceptors>
        <bean class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
    </jaxws:outInterceptors>
</jaxws:server>
```


```
命令空间：xmlns:jaxws="http://cxf.apache.org/jaxws"
schemaLocation：http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
```

部署并访问

```
http://localhost:8080/springTest/cxf/hello?wsdl
```

### jaxws:endpoint方式发布

applicationContext.xml文件

```
<!-- webservice注解类,可以是service层类 -->
<bean id="webServiceImpl" class="org.ipph.web.webservice.WebServiceImpl" />

<!-- 让CXF 与Spring整合,通过Spring发布 WS服务 -->
<!-- JAXWS 来发布WS服务 -->
<!-- address:就是此服务的访问地址:  http://localhost:8080/ws/cxf/hello  -->
<jaxws:endpoint    
    implementor="#webServiceImpl"
    address="/helloworld" >
    <jaxws:inInterceptors>  
        <bean class="org.apache.cxf.interceptor.LoggingInInterceptor" />
    </jaxws:inInterceptors>
    <jaxws:outInterceptors>
        <bean class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
    </jaxws:outInterceptors>
</jaxws:endpoint>
```

部署并访问

```
http://localhost:8080/springTest/cxf/helloworld?wsdl
```

### server与endpoint的区别

First, they are all for the server side configuration. 
Second, jaxws:endpoint is coming from JAXWS API, and it is used to configure the org.apache.cxf.jaxws.EndpointImpl which extends javax.xml.ws.Endpoint. 
jaxws:server is for configuring the JaxWsServerFactoryBean, which is coming from the Xfire API.

For the user who has the Xfire using experience, I think they will prefer to use jaxws:server tag. 
For the JAXWS API fans, jaxws:endpoint will be they first choice. 
There are not much difference between the jaxws:endpoint and jaxws:server, since the EndpointImpl is a wrapper class for the JaxWsServerFactoryBean.

另外，访问wsdl地址设置不同endpoint可以直接设，server这要根据你的项目及cxf.xml决定

```
<jaxws:endpoint    
    implementor="#webServiceImpl"
    address="http://localhost:7777/hello" >
    <jaxws:inInterceptors>  
        <bean class="org.apache.cxf.interceptor.LoggingInInterceptor" />
    </jaxws:inInterceptors>
    <jaxws:outInterceptors>
        <bean class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
    </jaxws:outInterceptors>
</jaxws:endpoint>
```

访问地址

```
http://localhost:7777/hello?wsdl
```

注：endpoint不推荐用，具体的话估计是wsdl一多，地址不好规范 ，但测试很方便 随便设地址



