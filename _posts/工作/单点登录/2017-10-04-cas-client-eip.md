---
title: 单点登录（eip系统）
tags: [cas]
---

eip系统结合spring security实现单点登录的原理分析。

1）修改配置文件

在app-context.xml中开启关于cas的配置

```
<import resource="app-security-cas.xml"/> 
```

2）相关jar包

```
WEB-INF/lib/cas-client-core-3.1.10.jar
WEB-INF/lib/spring-security-cas-client-3.0.3.RELEASE.jar
```

3）配置文件

```
<security:http pattern="/js/**" security="none" />
<security:http pattern="/commons/**" security="none" />
<security:http pattern="/media/**" security="none" />
<security:http pattern="/themes/**" security="none" />
<security:http pattern="/403*" security="none" />
<security:http pattern="/404*" security="none" />
<security:http pattern="/500*" security="none" />

<!-- 配置登录入口点 -->
<security:http entry-point-ref="casAuthenticationEntryPoint" auto-config="true"
    access-denied-page="/commons/403.jsp" 
    servlet-api-provision="true">
    
    <security:custom-filter  ref="aopFilter" position="FIRST" />
    
    <!-- cas过滤器-->
    <security:custom-filter position="CAS_FILTER" ref="casAuthenticationFilter" />

    <!-- spring单点退出过滤器-->
    <security:custom-filter before="LOGOUT_FILTER" ref="requestSingleLogoutFilter" />   

    <!-- cas单点退出过滤器-->                           
    <security:custom-filter before="CAS_FILTER" ref="singleLogoutFilter"/> 
    
    <!-- 要权限过滤器-->
    <security:custom-filter before="FILTER_SECURITY_INTERCEPTOR" ref="permissionFilter" />
    
</security:http>

<!-- 用于相同线程间共享Request对象，存储成ThreadLocal变量 -->
<bean id="aopFilter" class="com.hotent.core.web.filter.AopFilter"></bean>

<!-- cas单点登出-->
<bean id="singleLogoutFilter" class="org.jasig.cas.client.session.SingleSignOutFilter"/>

<!-- spring单点退出过滤器，并指定cas单点登出地址-->
<bean id="requestSingleLogoutFilter" 
      class="org.springframework.security.web.authentication.logout.LogoutFilter">
   <constructor-arg>
        <list>
            <bean class="org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler"/>
        </list>
    </constructor-arg>
    <!-- 单点退出后的返回页面-->
    <constructor-arg value="${cas.url}/logout?service=${platform.url}"/>
    <!-- 监听退的的url-->
    <property name="filterProcessesUrl" value="/logout" />
</bean>

<!-- 单点登陆过程，配置认证管理器-->
<bean id="casAuthenticationFilter"
    class="org.springframework.security.cas.web.CasAuthenticationFilter">
    <property name="authenticationManager" ref="authenticationManager" />
    <property name="authenticationSuccessHandler" ref="simpleUrlAuthenticationSuccessHandler"/>  
</bean>
<!-- 登陆成功后的跳转-->
<bean id="simpleUrlAuthenticationSuccessHandler"  
    class="org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler">
    <property name="alwaysUseDefaultTargetUrl" value="false"/>
    <property name="defaultTargetUrl" value="${platform.homepage}"/>  
</bean>

<!-- 认证管理器-->
<security:authentication-manager alias="authenticationManager">
    <security:authentication-provider ref="casAuthenticationProvider" />
</security:authentication-manager>

<!-- cas认证代理-->
<bean id="casAuthenticationProvider"
    class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
    <property name="userDetailsService" ref="sysUserDao" />  
    <property name="serviceProperties" ref="serviceProperties" />
    <property name="ticketValidator">
        <bean class="org.jasig.cas.client.validation.Cas20ServiceTicketValidator">
            <constructor-arg index="0" value="${cas.url}" />
        </bean>
    </property>
    <property name="key" value="an_id_for_this_auth_provider_only" />
</bean>

<!-- 本地spring认证类-->
<bean id="casAuthenticationUserDetailsService"
    class="org.springframework.security.core.userdetails.UserDetailsByNameServiceWrapper">
    <property name="userDetailsService" ref="sysUserDao" />
</bean>
<!-- 本地认证入口-->
<bean id="serviceProperties" class="org.springframework.security.cas.ServiceProperties">
    <property name="service" value="${platform.url}/j_spring_cas_security_check" /><!-- 本地认证入口-->
    <property name="sendRenew" value="false" />
</bean>

<!-- cas登陆入口--> 
<bean id="casAuthenticationEntryPoint"
    class="org.springframework.security.cas.web.CasAuthenticationEntryPoint">
    <property name="loginUrl" value="${cas.url}" /><!--cas 入口-->
    <property name="serviceProperties" ref="serviceProperties" />
</bean>

<!-- 权限过滤-->
<bean id="permissionFilter" class="com.hotent.core.web.filter.PermissionFilter">
    <property name="authenticationManager" ref="authenticationManager" />
    <property name="accessDecisionManager" ref="accessDecisionManager" />
    <property name="securityMetadataSource" ref="securityMetadataSource" />
</bean>
<!-- 跳转判断-->
<bean id="accessDecisionManager" class="com.hotent.platform.web.filter.HtDecisionManager" />
<!-- 权限资源-->
<bean id="securityMetadataSource"
    class="com.hotent.platform.web.filter.HtSecurityMetadataSource" scope="singleton">
    <property name="anonymousUrls">
        <set>
            <value>/login.ht</value>
            <value>/login.jsp</value>
            <value>/platform/system/sysFile/fileUpload.ht</value>
            <value>/platform/bpm/bpmDefinition/getXmlImport.ht</value>
        </set>
    </property>
</bean>
```

### 分析认证过程：

1）访问系统：

访问cas客户端：http://localhost:8080/eip-saas

响应信息302（重定向），location信息为：

```
http://localhost:8080/cas-server/?service=http://localhost:8080/eip-saas/j_spring_cas_security_check
```

依然响应302（重定向），location信息为：

```
http://localhost:8080/cas-server/login?service=http://localhost:8080/eip-saas/j_spring_cas_security_check
```

执行过程：

即由casAuthenticationEntryPoint设置的登录入口来处理，处理结果是将请求重定向到一个url上，这个url的组成为serviceProperties中设置的service信息。

```
<bean id="serviceProperties" class="org.springframework.security.cas.ServiceProperties">
    <property name="service" value="${platform.url}/j_spring_cas_security_check" /><!-- 本地认证入口-->
    <property name="sendRenew" value="false" />
</bean>
```

查看app.properties可以发现platform.url为http://localhost:8080/eip-saas/。

2）登录

点击登录按钮，响应302，即重定向会cas客户端，并将ticket作为参数返回

location地址：

```
http://localhost:8080/eip-saas/j_spring_cas_security_check?ticket=ST-1-GYrKdfKHvmh2ySfG69Ps-cas01.example.org
```

在web.xml中配置了springSecurityFilterChain的拦截信息，因此该重定向路径会被security拦截到。

```
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/j_spring_cas_security_check</url-pattern>
</filter-mapping>
```

在security中处理时，会交由permissionFilter权限过滤器来处理。

3）客户端的认证和授权

这里使用permissionFilter来进行用户认证的，判断客户端是否包含该用户，如果包含则该用户登录成功，如果不包含则提示用户不存在。

```
<!-- 要权限过滤器-->
<security:custom-filter before="FILTER_SECURITY_INTERCEPTOR" ref="permissionFilter" />
<bean id="permissionFilter" class="com.hotent.core.web.filter.PermissionFilter">
    <property name="authenticationManager" ref="authenticationManager" />
    <property name="accessDecisionManager" ref="accessDecisionManager" />
    <property name="securityMetadataSource" ref="securityMetadataSource" />
</bean>
```

查看authenticationManager的实现

```
<!-- 认证管理器-->
<security:authentication-manager alias="authenticationManager">
    <security:authentication-provider ref="casAuthenticationProvider" />
</security:authentication-manager>

<!-- cas认证代理-->
<bean id="casAuthenticationProvider"
    class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
    <property name="userDetailsService" ref="sysUserDao" />  
    <property name="serviceProperties" ref="serviceProperties" />
    <property name="ticketValidator">
        <bean class="org.jasig.cas.client.validation.Cas20ServiceTicketValidator">
            <constructor-arg index="0" value="${cas.url}" />
        </bean>
    </property>
    <property name="key" value="an_id_for_this_auth_provider_only" />
</bean>
```

注：认证的过程交由casAuthenticationProvider来实现，可以查看这部分的源代码。可以发现，主要是通过UserDetailService的实现类来查询cas-client端的用户信息，来判断客户端是否存在与服务器对应的用户信息。

4）casAuthenticationProvider源码

```
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    ...

    if (result == null) {
        result = this.authenticateNow(authentication);
        result.setDetails(authentication.getDetails());
    }

    if (stateless) {
        // Add to cache
        statelessTicketCache.putTicketInCache(result);
    }
    return result;
}

//完成认证信息
private CasAuthenticationToken authenticateNow(final Authentication authentication) throws AuthenticationException {
    try {
        //校验ticket是否正确，与serviceProperties设置的service比对
        final Assertion assertion = this.ticketValidator.validate(authentication.getCredentials().toString(), serviceProperties.getService());
        //获取cas认证的用户信息（从本地获取，通过cas中填写的用户名），重点
        final UserDetails userDetails = loadUserByAssertion(assertion);

        ...
    } catch (final TicketValidationException e) {
        throw new BadCredentialsException(e.getMessage(), e);
    }
}
//获取认证用户
protected UserDetails loadUserByAssertion(final Assertion assertion) {
    final CasAssertionAuthenticationToken token = new CasAssertionAuthenticationToken(assertion, "");
    //加载用户
    return this.authenticationUserDetailsService.loadUserDetails(token);
}
```

注：这里的authenticationUserDetailsService类被UserDetailsByNameServiceWrapper包装了一下。

```
public void setUserDetailsService(final UserDetailsService userDetailsService) {
    this.authenticationUserDetailsService = new UserDetailsByNameServiceWrapper(userDetailsService);
}
```

注：实际上会调用sysUserDao类

```
<bean id="casAuthenticationProvider"
    class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
    <property name="userDetailsService" ref="sysUserDao" />  
    ...
</bean>
//类定义信息，实现了userDetailsService接口
SysUserDao extends BaseDao<SysUser> implements UserDetailsService
```

用户的获取是通过调用SysUserDao类的loadUserByUsername方法

```
@Override
public UserDetails loadUserByUsername(String username)
        throws UsernameNotFoundException, DataAccessException {
    SysUser sysUser = getByAccount(username);
    if (sysUser == null)
        throw new UsernameNotFoundException("用户不存在");
    return sysUser;
}
```

5）cas服务器登录的用户信息是如何传递到cas客户端的？

主要是通过再次查询的方式获取用户的具体信息。这里只是获取了用户的名称。

```
final Assertion assertion = this.ticketValidator.validate(authentication.getCredentials().toString(), serviceProperties.getService());
```

Cas20ServiceTicketValidator类的validate方法，实现方法位于父类AbstractUrlBasedTicketValidator中

```
public Assertion validate(final String ticket, final String service) throws TicketValidationException {

    final String validationUrl = constructValidationUrl(ticket, service);

    try {
        final String serverResponse = retrieveResponseFromServer(new URL(validationUrl), ticket);

        if (serverResponse == null) {
            throw new TicketValidationException("The CAS server returned no response.");
        }

        return parseResponseFromServer(serverResponse);
    } catch (final MalformedURLException e) {
        throw new TicketValidationException(e);
    }
}
```

注：构造出cas-server的验证url地址，再次发送请求获取ticket对应的用户信息。

Cas20ServiceTicketValidator类解析获取的响应

```
protected final Assertion parseResponseFromServer(final String response) throws TicketValidationException {
    ...
    //获取用户信息
    final String principal = XmlUtils.getTextForElement(response, "user");
    
    ...
    final AttributePrincipal attributePrincipal = new AttributePrincipalImpl(principal, proxyGrantingTicket, this.proxyRetriever);
        assertion = new AssertionImpl(attributePrincipal);
    ...
    return assertion;
}
```