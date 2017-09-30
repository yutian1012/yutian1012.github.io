---
title: 单点登录（服务器端）
tags: [cas]
---

参考：http://blog.csdn.net/lifetragedy/article/details/43817903

SSO即Single Sign On（单点登录）。

场景：在多个系统环境下，用户只登录一次，就可在多个系统间来回切换，并且不需要重新认证。

### CAS产品

CAS是SSO的一个实现，基于spring配置来实现，分为server版和client版两个模块。同时对于一些其它功能特性如：数据库、LDAP协议（就是WINDOWS-AD域使用的协议）、安全等还提供了一系列的插件。

下载地址：https://www.apereo.org/projects/cas/download-cas

![](/images/work/cas/server-client.jpg)

### 部署CAS服务器

下载cas-server后，解压缩文件从modules目录下拷贝cas-server-webapp-3.5.2.war到tomcat服务器下，并启动tocmat。

访问http://localhost:8080/cas-server-webapp-3.5.2/，会显示认证服务器的登录页面

### 编辑CAS认证用户的数据源

cas服务是使用spring容器来管理的，因此修改配置时只需要修改spring配置文件即可。修改cas-server\WEB-INF\deployerConfigContext.xml文件

```
//authenticationManager类的authenticationHandlers属性中，注释掉下面的认证处理
<!-- <bean class="org.jasig.cas.authentication.handler.support.SimpleTestUsernamePasswordAuthenticationHandler" /> -->
//在该属性下添加数据源认证
<bean class="org.jasig.cas.adaptors.jdbc.QueryDatabaseAuthenticationHandler">  
      <property name="dataSource" ref="dataSource" ></property>  
      <property name="sql" value="select password from sys_user where user_id=?" ></property>
</bean> 
...
//添加数据源
<bean id="dataSource"  
    class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />  
    <property name="url" value="jdbc:mysql://172.17.30.65:3306/cas" />  
    <property name="username" value="root" />  
    <property name="password" value="xxx" />  
</bean>  
```

创建数据表（mysql）

```
CREATE TABLE SYS_USER  
(  
    USER_ID VARCHAR(16),   
    PASSWORD VARCHAR(8),   
    CONSTRAINT PK_SYS_USER PRIMARY KEY (USER_ID)  
);

insert into sys_user values('sso','aaaaaa');
```

注：运行前需要把相应的jar包拷贝到lib目录下，拷贝cas-server-support-jdbc-3.5.2.jar，mysql-connector-java-5.1.27。

访问服务，登录时用户名为sso，密码为aaaaaa，即可以看到正确的授权信息。

### 不使用HTTPS认证

CAS SSO严格意义上来说需要J2EE APP SERVER里实现HTTPS即SSL的双向认证模式才能正常使用，为此，我们要关闭CAS的HTTPS认证模式。

修改：tomcat\webapps\cas-server\WEB-INF\spring-configuration目录下的ticketGrantingTicketCookieGenerator.xml文件

```
<bean id="ticketGrantingTicketCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
        p:cookieSecure="true"
        p:cookieMaxAge="-1"
        p:cookieName="CASTGC"
        p:cookiePath="/cas" />
```

把这边的p:cookieSecure从true改为false即可。

### 补充：Windows系统AD域实现与系统的单点登录

用户的AD域帐户就是它的系统帐户。