---
title: spring security快速构建测试项目
tags: [spring security]
---

实例：大部分的项目中安全处理是通过RBAC（基于角色的访问控制）来实现的。即访问应用不同资源需要不同角色，这里通过实例学习如何通过使用springsecurity来进行项目安全方面的设计和实现。

参考：http://blog.csdn.net/AirMario/article/details/53955727

### 构建测试项目

1）创建maven的web项目

在maven项目中创建两个jsp文件，用来实现不同权限访问不同页面的需求。

admin.jsp文件，只有拥有ROLE_ADMIN角色的用户才能访问该页面。

index.jsp，拥有ROLE_USER或ROLE_ADMIN角色的用户都能访问该页面。

注：admin.jsp页面比index.jsp页面需要更高的权限

2）引入spring security相关依赖

修改pom.xml文件

```
<properties>  
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
    <java-version>1.7</java-version>  
    <org.springframework-version>4.3.13.RELEASE</org.springframework-version>
    <spring-security-version>4.2.3.RELEASE</spring-security-version>
</properties> 

<dependencies>
 <!-- Spring -->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-context</artifactId>  
        <version>${org.springframework-version}</version>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-webmvc</artifactId>  
        <version>${org.springframework-version}</version>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-context-support</artifactId>  
        <version>${org.springframework-version}</version>  
    </dependency>  
    <!-- Spring security -->  
    <dependency>  
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>  
        <version>${spring-security-version}</version>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework.security</groupId>  
        <artifactId>spring-security-config</artifactId>  
        <version>${spring-security-version}</version>  
    </dependency>  
    <dependency>  
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
        <version>4.12</version>  
        <scope>test</scope>  
    </dependency>  
</dependencies>

<build>
    <finalName>springsecuritydemo</finalName>
    <pluginManagement>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>${java-version}</source>
                    <target>${java-version}</target>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

3）配置spring容器

在resources中新建applicationContext.xml文件，并编辑内容

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans:beans xmlns="http://www.springframework.org/schema/security"  
    xmlns:beans="http://www.springframework.org/schema/beans"   
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd  
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-4.3.xsd  
                        http://www.springframework.org/schema/security
                        http://www.springframework.org/schema/security/spring-security-4.2.xsd">  
    <http auto-config='true'>  
        <!-- 定义拦截资源，以及资源需要的角色信息 -->
        <intercept-url pattern="/admin.jsp" access="hasRole('ROLE_ADMIN')" />  
        <intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>  
    </http>  
    <authentication-manager>  
        <authentication-provider>  
            <user-service>  
                <!-- 配置的方式定义用户，以及用户所授予的角色信息 -->
                <user name="admin" password="123" authorities="ROLE_ADMIN,ROLE_USER" />  
                <user name="user" password="123" authorities="ROLE_USER" />  
            </user-service>  
        </authentication-provider>  
    </authentication-manager>
</beans:beans>
```

4）修改web.xml

```
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"  
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee  
     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
    version="3.0">  
    <display-name>SpringSecurity</display-name>  
      
    <!-- 加载配置文件 -->  
    <context-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:applicationContext.xml</param-value>  
    </context-param>  
    <!-- spring security 的过滤器配置 -->  
    <filter>  
        <filter-name>springSecurityFilterChain</filter-name>  
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
    </filter>  
    <filter-mapping>  
        <filter-name>springSecurityFilterChain</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
  
    <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  
      
</web-app>  
```

5）测试访问

当直接访问应用时，由于没有登录会直接跳转到登录页面，该页面不需要手动创建，spring security会自动创建。

访问index.jsp页面：

```
http://localhost:8080/springsecuritydemo/
```

输入用户user并输入密码123，就能正常访问index.jsp页面。

访问admin.jsp页面：

```
http://localhost:8080/springsecuritydemo/admin.jsp
```

如果使用user用户登录，则会拒绝用户访问（403 Access is denied）。清空缓存浏览器缓存，再次访问，又会提示登录。这次使用admin登录，然后可以正常访问admin.jsp页面。

### 思考：

通过这个简单的项目我们能够发现：

springsecurity拦截请求路径，要求用户提供认证信息（通过登录的方式会获取用户的信息）；通过用户信息获取用户具有的角色，再根据角色信息判断是否能够访问该请求；如果能够访问则正常返回页面，否则返回403。

这里就涉及到了：用户，角色和资源（拦截路径）的概念。

1）用户

用户这里我们是直接配置在applicationContext.xml中的，配置时为用户提供了用户名，密码和拥有的角色信息。那么，进一步考虑，用户信息可以通过存储在数据库中来提供，用户所拥有的角色也可以结合原系统中用户角色的关系来获取。

2）角色

角色也是在applicationContext.xml中配置的，是以ROLE_开头的字符串。除此之外，可以看到与角色相关的地方主要有用户和资源访问路径两个地方，这里角色起到了桥梁的作用。

3）资源（拦截路径）

资源是用户访问的信息，来源于用户的访问。而在applicationContext.xml中规定了访问资源所必须具备的角色，来实现访问控制。