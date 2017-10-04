---
title: 单点登录（LDAP，cas-server）
tags: [cas]
---

在openldap中创建了用户user1，密码为6个1，现在使用CAS结合LDAP进行授权登录。即CAS-SSO单点登录中的用户名从原来的数据库改变为ldap里的用户名和密码。

1）在cas-server中引用ldap相关的jar包

```
cas-server-support-ldap-3.5.2.jar
spring-context-3.1.0.RELEASE.jar
spring-context-support-3.1.0.RELEASE.jar
spring-ldap-core-1.3.0.RELEASE.jar
spring-ldap-core-tiger-1.3.0.RELEASE.jar
spring-context-3.1.0.RELEASE.jar
spring-context-support-3.1.0.RELEASE.jar
spring-binding-2.3.0.RELEASE.jar
```

注：cas-server-support-ldap-3.5.2.jar在cas-server3.5.2的modules目录下去查找。

2）配置ldap数据源

修改cas-server\WEB-INF目录下的deployerConfigContext.xml文件

添加Ldap数据源

```
<bean id="contextSource" class="org.springframework.ldap.core.support.LdapContextSource">  
      <property name="password" value="secret" />  
      <property name="pooled" value="true" />  
      <property name="url" value="ldap://localhost:389"/>  
     
      <!--管理员 -->  
      <property name="userDn" value="cn=Manager,dc=maxcrc,dc=com" />  
      <property name="baseEnvironmentProperties">  
         <map>  
               <!-- Three seconds is an eternity to users. -->  
              <entry key="com.sun.jndi.ldap.connect.timeout" value="60" />  
              <entry key="com.sun.jndi.ldap.read.timeout" value="60" />  
                    <entry key="java.naming.security.authentication" value="simple" />  
         </map>  
    </property>  
</bean> 
```

配置handler，在authenticationHandlers中添加配置

```
<bean class="org.jasig.cas.adaptors.ldap.BindLdapAuthenticationHandler">  
      <property name="filter" value="cn=%u" />  
      <!-- 基节点 -->  
      <property name="searchBase" value="ou=members,o=company,dc=maxcrc,dc=com" />  
      <property name="contextSource" ref="contextSource" />  
</bean>
```

注：contextSource引用配置的Ldap数据源。

3）分析

LDAP里是怎么寻找cn=user1的连接地址？

cn=user1,ou=members,o=company,dc=sky,dc=org。配置中以ou=members,o=company,dc=sky,dc=org为基本地址（searchbase)，寻找下面一切cn=为开头的用户名是否匹配（filter)，如果用户名匹配了，那么cas会自动从LDAP里把该用户的userDn取出来，同时索取该用户所绑定的相关密码。

注：这里的filter值为cn=%u。

4）测试

使用部署的客户端来访问，会跳转到cas-server认证服务器上，输入user1，密码6个1能成功登录。

```
http://localhost:9080/cas-sample-site2/
```