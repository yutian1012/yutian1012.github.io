---
title: 开源spagobi--SpagoBIProject项目源码分析
tags: [spagobi]
---

该项目是一个maven管理的web项目，提供bi平台的展示和管理功能，同时也是用户登录系统的入口。

### 1. ProfileFilter过滤器

从SessionContainer中获取Profile对象，profile对象中包含了用户信息，角色信息以及权限等信息。

注：该过滤器的主要是将Profile对象放置到SessionContainer中，并将Tenant对象设置到TenantManager线程局部变量中，Tenant对象用户的组织（Organization）。

### 2. AntiInjectionFilter过滤器

注：实现防XSS攻击（跨站脚本攻击），即通过替换request中参数的特殊字符，防止攻击。

### 3. QuartzInitializer

```
<servlet>
    <servlet-name>QuartzInitializer</servlet-name>
    <servlet-class>org.quartz.ee.servlet.QuartzInitializerServlet</servlet-class>     
    <init-param>
        <param-name>shutdown-on-unload</param-name>
        <param-value>true</param-value>
    </init-param>  
    <init-param>
        <param-name>start-scheduler-on-load</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>      
</servlet>
```

注：初始化Quartz定时器。

### 3. AdapterHTTP实现方法映射

该类与spring mvc中的DispatchServlet相似，实现请求映射到相应的方法。
