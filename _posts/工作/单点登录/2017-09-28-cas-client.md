---
title: 单点登录（客户端）
tags: [cas]
---

参考：http://blog.csdn.net/lifetragedy/article/details/43817903

### 客户端下载

1）下载源码

下载地址：https://github.com/apereo/java-cas-client

在git中选择tags/3.2.1，然后点击右侧的下载压缩包（源码）。需要在命令行下将其打包成jar文件。

2）执行命令：

```
mvn clean package
```

注：在下载atlassian-seraph.jar包时，由于仓库无法找到，需要在相应的pom中配置repository地址。

3）修改pom文件

修改：java-cas-client-3.2.1\cas-client-integration-atlassian\pom

```
<repositories>
    <repository>
        <id>atlassian</id>
        <name>Atlassian Repository</name>
        <url>https://maven.atlassian.com/content/repositories/atlassian-public/</url>
    </repository>
    <repository>
        <id>atlassian2</id>
        <name>Atlassian2 Repository</name>
        <url>https://maven.atlassian.com/3rdparty/</url>
    </repository>
</repositories>
```

修改：java-cas-client-3.2.1\cas-client-support-distributed-memcached\pom.xml

```
<dependency>
    <groupId>spy</groupId>
    <artifactId>memcached</artifactId>
    <version>2.5</version>
    <type>jar</type>
    <scope>system</scope>
    <systemPath>G:/repository/spy/memcached/2.5/memcached-2.5.jar</systemPath>
</dependency>
```

运行完成后，会在java-cas-client-3.2.1\cas-client-core\target目录下生成cas-client-core-3.2.1.jar包。

注：该jar包可以直接从mvn中获取。

### 创建测试案例

在eclipse中创建web项目cas-sample-site1

1）引入jar包

在lib中引入cas-client-core-3.2.1.jar包，

2）创建index.jsp页面。

```
<body>  
    <h1>cas sample site1</h1>  
      
    <a href="http://localhost:9080/cas-sample-site1/index.jsp">
        cas-sample-site2
    </a>  
      
    </br>
    <a href="http://localhost:8080/cas-server/logout">退出</a> 
</body>
```

注：这里cas-server启动的端口是8080，客户端使用的是9080

3）编辑web.xml文件

```
<listener>
    <listener-class>
        org.jasig.cas.client.session.SingleSignOutHttpSessionListener
    </listener-class>  
</listener>  

<!--退出过滤器-->
<filter>  
    <filter-name>CAS Single Sign Out Filter</filter-name>  
    <filter-class>
        org.jasig.cas.client.session.SingleSignOutFilter
    </filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>CAS Single Sign Out Filter</filter-name>  
    <url-pattern>*</url-pattern>
</filter-mapping>  

<filter>  
    <filter-name>CAS Validation Filter</filter-name>  
    <filter-class>org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>  
    <init-param>  
        <!-- cas server的服务器所在的地址 -->
        <param-name>casServerUrlPrefix</param-name>  
        <param-value>http://localhost:8080/cas-server</param-value>  
    </init-param>  
    <init-param>  
        <!-- 子系统的服务地址 -->
        <param-name>serverName</param-name>  
        <param-value>http://localhost:9080</param-value>  
    </init-param>  
    <init-param>  
        <param-name>useSession</param-name>  
        <param-value>true</param-value>  
    </init-param>  
    <init-param>  
        <param-name>redirectAfterValidation</param-name>  
        <param-value>true</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>CAS Validation Filter</filter-name>  
    <url-pattern>*</url-pattern>  
</filter-mapping>  

<filter>  
    <filter-name>CAS Filter</filter-name>  
    <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>  
    <init-param>  
        <param-name>casServerLoginUrl</param-name>  
        <param-value>http://localhost:8080/cas-server/login</param-value>  
    </init-param>  
    <init-param>  
        <param-name>serverName</param-name>  
        <param-value>http://localhost:9080</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>CAS Filter</filter-name>  
    <url-pattern>*</url-pattern>  
</filter-mapping>  

<filter>  
    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
    <filter-class>org.jasig.cas.client.util.HttpServletRequestWrapperFilter</filter-class>  
</filter>  
<filter-mapping>  
    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>  
    <url-pattern>*</url-pattern>  
</filter-mapping>
```

注：配置了1个listener和3个filter。

### 测试客户端

将客户端web应用部署到jboss或其他servlet服务器上，访问客户端会自动跳转到cas-server认证服务器上。登录后，又重新回到客户端服务上。

### 测试多个客户端能否实现单点登录

在eclipse中创建web项目cas-sample-site2，与上面的创建方式相同，内容也基本相同。

在客户端cas-sample-site1上登录，然后访问cas-sample-site2，发现cas-sample-site2不用再次登录了。并且，从任何一个客户端服务器上退出后其他服务器都不能再访问了。