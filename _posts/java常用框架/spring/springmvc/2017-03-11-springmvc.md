---
title: spring mvc环境搭建
tags: [spring]
---

搭建spring和spring mvc环境

首先创建一个maven的web项目

### 1. 在maven项目中引入相关jar包

```
<properties>
    <spring.version>4.2.5.RELEASE</spring.version>
    <log4j.version>1.2.17</log4j.version>
    <jstl.version>1.2</jstl.version>
</properties>

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

<!-- 编译环境 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

### 2. 配置框架

1）配置spring环境

web.xml文件：

```
<context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:applicationContext.xml</param-value>
</context-param>
  
<!-- Spring监听 -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

在resources目录下创建applicationContext.xml文件：

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

2）配置spring mvc环境

web.xml文件：

```
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
```

在resources目录下创建springMVC-servlet.xml文件：

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
    <context:component-scan base-package="org.ipph.web.controller" />
    
    <!-- 定义跳转的文件的前后缀 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
        <property name="prefix" value="/WEB-INF/jsp/" />  
        <property name="suffix" value=".jsp" />
    </bean>  
</beans>
```

### 3. 测试框架整合

1）构造controller，新建包org.ipph.web.controller包，新建类HelloworldController

```
@Controller
@RequestMapping("/ipph/hello")
public class HelloworldController {
    @RequestMapping("/test1")
    public ModelAndView helloworld(HttpServletRequest request,HttpServletResponse response){
        return new ModelAndView("/ipph/hello/Helloworld");
    }
}
```

注：请求路径为/ipph/hello/test1

注2：在springMVC-servlet.xml文件中配置扫描路径到controller类目录上。

2）创建jsp页面

在WEB-INF下创建目录/jsp/ipph/hello，并在hello目录下创建Helloworld.jsp文件

3）部署运行测试

访问地址：http://localhost:8080/webmodel/ipph/hello/test1，正确访问到了Helloworld.jsp页面

### 4. spring mvc的请求和转发功能

```
@Controller
@RequestMapping("/ipph/hello")
public class HelloworldController {
    @RequestMapping("/test1")
    public ModelAndView helloworld(HttpServletRequest request,HttpServletResponse response){
        return new ModelAndView("/ipph/hello/Helloworld");
    }
    
    @RequestMapping("/test2")
    public String helloworld2(HttpServletRequest request,HttpServletResponse response){
        return "/ipph/hello/Helloworld";
    }
    
    //重定向测试
    @RequestMapping("/test3")
    public String helloworld3(HttpServletRequest request,HttpServletResponse response){
        return "redirect:/ipph/hello/test1";
    }
    
    @RequestMapping("/test4")
    public ModelAndView helloworld4(HttpServletRequest request,HttpServletResponse response){
        return new ModelAndView("redirect:/ipph/hello/test1");
    }
}
```
