---
title: 微信java开发环境搭建
tags: [weixin]
---

1）创建maven项目（web），并导入依赖

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.ipph.app</groupId>
  <artifactId>weixin</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>weixin Maven Webapp</name>
  <url>http://maven.apache.org</url>
  
  <properties>
    <spring.version>4.2.5.RELEASE</spring.version>
    <log4j.version>1.2.17</log4j.version>
    <jstl.version>1.2</jstl.version>
    <dom4j.version>1.6.1</dom4j.version>
    <Gson.version>2.8.0</Gson.version>
  </properties>
  
  <dependencies>
    <!-- spring框架 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    
    <!-- 标签 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>${jstl.version}</version>
    </dependency>
        
    <!-- 日志框架 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>
    
    <!-- 结合第三方的业务会用到json的解析-->
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>${Gson.version}</version>
    </dependency>
        
    <!-- 编译环境 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>
    
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>${dom4j.version}</version>
    </dependency>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>weixin</finalName>
    <pluginManagement>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
            </configuration>
        </plugin>
    </plugins>
   </pluginManagement>
</build>
</project>
```

2）spring mvc的配置

a.web.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>  
<web-app  
        version="3.0"  
        xmlns="http://java.sun.com/xml/ns/javaee"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"> 
        
  <display-name>Archetype Created Web Application</display-name>
  
    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
      
    <!-- Spring监听 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    
    <filter>  
       <filter-name>characterEncodingFilter</filter-name>  
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
       <init-param>  
           <param-name>encoding</param-name>  
           <param-value>UTF-8</param-value>  
       </init-param>  
       <init-param>  
           <param-name>forceEncoding</param-name>  
           <param-value>true</param-value>  
       </init-param>  
   </filter>  
   <filter-mapping>  
       <filter-name>characterEncodingFilter</filter-name>  
       <url-pattern>/*</url-pattern>  
   </filter-mapping>  
    
    <!-- Spring MVC DispatcherServlet -->
    <servlet>
        <servlet-name>springMVC3</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springMVC-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- 配置拦截路径 -->
    <servlet-mapping>
        <servlet-name>springMVC3</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

b.spring配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
           http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.2.xsd">
</beans>
```

c.spring mvc相关的配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mvc="http://www.springframework.org/schema/mvc" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
           http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd "
       default-autowire="byName" default-lazy-init="false">
 
    <!-- 添加注解驱动 -->  
    <mvc:annotation-driven />
    
    <!-- 默认扫描的包路径 -->  
    <context:component-scan base-package="org.ipph.app" />
    
    <!-- 定义跳转的文件的前后缀 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
        <property name="prefix" value="/WEB-INF/jsp/" />  
        <property name="suffix" value=".jsp" />
    </bean>  
</beans>
```

3）环境测试

a.创建测试类

```
package org.ipph.app.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
@RequestMapping("/test")
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView helloworld(HttpServletRequest request,HttpServletResponse response){
        return new ModelAndView("/test/hello");
    }
}
```

b.创建jsp页面

在webapp/WEB-INF/jsp/test目录下创建hello.jsp

c.部署并测试

```
localhost:8080/weixin/test/hello
```