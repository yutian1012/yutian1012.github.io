---
title: spring security快速构建测试项目分析
tags: [spring security]
---

分析：applicationContext.xml文件的配置信息

```
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
```

1）明确安全拦截器的作用位置？

2）http标签的auto-config做了哪些工作？

3）authentication-manager标签的功能？

