---
title: ruisiBI标准版源代码查看
tags: [BI]
---

官网下载的标准版通过反编译工具jg-gui反编译包ruisi-bi.jar和ext3-bi.jar来查看源码架构。

### 代码框架

使用spring+struts2+mybatis编写的。

1）spring配置文件

位于WEB-INF\classes\spring.xml，该文件主要实现了数据源，属性文件等配置信息。

2）struts2配置文件

位于WEB-INF\classes\struts.xml，并且权限设置是通过struts2的拦截器来实现的。

配置文件的解释

```
<!-- 减轻struts的result标签的配置，简化页面跳转设置 -->
<!--  
struts.codebehind.pathPrefix  = /jsp/
package namespace = /
action name = userlist
action returntype = 为success时，值为空，为其他时，值为"-" + return type
返回的页面地址为：/jsp/userlist.jsp或/jsp/userlist-input.jsp（如果return type为input）
-->
<!-- 添加依赖struts2-codebehind-plugin-2.0.11.2.jar包到应用中 -->
<constant name="struts.codebehind.pathPrefix" value="/pages/" />

<!-- 指定系统中的action的默认package -->
<constant name="struts.configuration.classpath.defaultParentPackage" value="base-default" />

<!-- action的访问路径是通过与namespace拼接形成的 -->
<package name="base-default" namespace="/" extends="struts-default">
    <interceptors>
        <interceptor name="authInterceptor" class="com.ruisi.vdop.service.frame.LoginCheckInterceptor" />
        <interceptor-stack name="boncDefaultStack">
            <interceptor-ref name="authInterceptor" />
            <interceptor-ref name="defaultStack" />
        </interceptor-stack>
    </interceptors>
    
    <default-interceptor-ref name="boncDefaultStack" />

    <global-results>
        <result name="noLogin">/pages/control/NoLogin.jsp</result>
        <result name="forbid">/pages/control/Forbid.jsp</result>
        <result name="dateout">/pages/Login-dateout.jsp</result>
    </global-results>
</package>
```

3）action类如何自动识别

在web.xml中定义了filter，同时用来加载struts的配置信息

```
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <init-param>
        <!-- 指定action的包路径，会自动扫描该包下的类，作为struts的action -->
        <param-name>actionPackages</param-name>
        <param-value>com.ruisi.vdop.web</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>*.action</url-pattern>
</filter-mapping>
```