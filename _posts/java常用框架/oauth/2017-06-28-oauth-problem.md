---
title: oauth常见错误
tags: [oauth]
---

### resource id值

1）错误授权信息

```
Invalid token does not contain resource id (cpquery-resource)
```

每个resource-id的值必须在对应的ClientDetails中resourceIds值中存在，即在表oauth_client_details表的resource_ids字段中。

注：用户在使用client_id和accesstoken时，会判断client_id对应的记录是否有相应的资源访问

2）配置文件

```
<!-- resource-server配置-每个resource-id的值必须在对应的ClientDetails中resourceIds值中存在 -->
<oauth2:resource-server id="cpqueryResourceServer" resource-id="cpquery-resource" token-services-ref="tokenServices"/>

拦截路径
<!-- resource server -->
<security:http pattern="/cpquery/**" create-session="never" entry-point-ref="oauth2AuthenticationEntryPoint" access-decision-manager-ref="oauth2AccessDecisionManager" use-expressions="false">  
    <security:anonymous enabled="false"/>
    <security:intercept-url pattern="/cpquery/**" access="ROLE_USER,SCOPE_READ"/>
    <!-- 应用定义的resource server -->
    <security:custom-filter ref="cpqueryResourceServer" before="PRE_AUTH_FILTER"/>
    <security:access-denied-handler ref="oauth2AccessDeniedHandler"/>
    <security:csrf disabled="true"/>
</security:http>
```

注：配置文件的第一部分是资源的判断，第二部分为拦截的路径，即如果访问该路径就需要进行资源拥有权的判断。