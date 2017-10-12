---
title: 单点登录（服务器端）
tags: [cas]
---

参考：http://blog.csdn.net/lifetragedy/article/details/43817903

SSO即Single Sign On（单点登录）。

场景：在多个系统环境下，用户只登录一次，就可在多个系统间来回切换，并且不需要重新认证。

### 场景实例

现有两个系统A和B，想要让A系统和B系统实现单点登录，引入了cas认证服务器。并且A和B系统都有各自的用户和权限。

cas认证服务器的用户信息：

1）与A系统共用用户（连接查询A系统数据库中的用户表）

从A系统的用户中获取，A系统中的用户即为A和B系统的用户综合（所有使用A、B系统的用户），在A系统中录入了用户信息，要同步到B系统中，使A系统和B系统用户数量和用户名一一对应。

注：在A或B系统中添加用户时需要同步用户到另一端的系统用户模块中。

2）cas使用独立的用户体系（这种方式更繁琐）

为CAS创建独立的用户，在添加用户时统一从CAS服务器所连接的用户库中添加（最好不要在A和B系统端添加用户），并同时向A和B系统中同步用户。

注：如果从A或B中添加用户，那么就需要同用户到另外一端的系统中，并同时同步到cas服务器的用户中。

同步用户信息：

1）同步用户的操作可以通过A和B系统提供相应的web-service接口来发送和接收同步信息的方式实现。

2）通过提供一个消息中间件，A和B系统都来监听相应的创建用户的消息，并在各种的系统中同步用户。

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

1）使用http协议存在的问题，不能实现多系统的单点登录

一般情况下用户使用http进行认证操作，系统A登录转到CAS服务器，CAS服务器登录后会将ticket等信息保存到cookie中（这时会限制cookie的读取，必须https方式访问CAS才能使用该cookie），然后重定向到系统A的指定回调url，并在url后追加相关参数ticket。系统A收到CAS服务器发送的请求信息，并获取CAS添加在url后的参数ticket，系统A处理登录时，再次向CAS请求认证并传递该ticket，cas获取该ticket发现用户已经认证，返回系统A相应的登录用户名，系统A能够实现登录。

这时在同一个浏览器中访问系统B，由于系统B也会访问CAS，即登录页会重定向到CAS服务器，由于重定向使用的是http协议，不能将cas保存的cookie信息（ticket）附加到请求url中，因此CAS不能识别出登录信息，造成系统B不能实现单点登录。

注：可以在浏览器中使用F12（调试控制台），查看系统A，系统B登录时，重定向到CAS服务器时，url访问传递的参数信息（有无携带相应的CAS保存的cookie信息）。

2）解决方法

CAS SSO严格意义上来说需要J2EE APP SERVER里实现HTTPS即SSL的双向认证模式才能正常使用，为此，我们要关闭CAS的HTTPS认证模式。即使其支持http协议认证。

修改：tomcat\webapps\cas-server\WEB-INF\spring-configuration目录下的ticketGrantingTicketCookieGenerator.xml文件

```
<bean id="ticketGrantingTicketCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
        p:cookieSecure="true"
        p:cookieMaxAge="-1"
        p:cookieName="CASTGC"
        p:cookiePath="/cas" />
```

把这边的p:cookieSecure从true改为false即可。