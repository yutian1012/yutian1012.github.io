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

web请求信息

2）http标签的auto-config做了哪些工作？

3）intercept-url属性的处理？

4）authentication-manager标签的功能？

### 安全拦截器的作用位置

参考：http://blog.csdn.net/u013828625/article/details/72636744

1）spring security的引入

在web.xml中配置了DelegatingFilterProxy，在使用该过滤器时，内部会转交spring容器设置的springSecurityFilterChain对象，该对象中包含一个过滤器链。即在请求时过滤器DelegatingFilterProxy执行doFilter方法，将转由springSecurityFilterChain对象来处理，处理过程就会经过内部的过滤器链来处理。

springSecurityFilterChain对象是在spring容器解析xml配置文档时，遇到了http标签，使用HttpSecurityBeanDefinitionParser类解析时，自动将springSecurityFilterChain设置到了容器中。

![](/images/springspringsecurity/filterChains.png)

```
<!-- spring security 的过滤器配置 -->  
<filter>  
    <filter-name>springSecurityFilterChain</filter-name>  
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
</filter>  
<!-- 拦截所有的请求路径 -->
<filter-mapping>  
    <filter-name>springSecurityFilterChain</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

2）DelegatingFilterProxy类的继承层次

DelegatingFilterProxy-->GenericFilterBean-->Filter，间接的基础类Filter接口。

拦截路径时会执行DelegatingFilterProxy类的doFilter方法进行拦截。

```
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
    throws ServletException, IOException {

    // Lazily initialize the delegate if necessary.
    Filter delegateToUse = this.delegate;
    if (delegateToUse == null) {
            ...
            //从spring容器中获取delegate
            //wac是WebApplicationContext的实例
            delegateToUse = initDelegate(wac);
        }
    }

    // Let the delegate perform the actual doFilter operation.
    invokeDelegate(delegateToUse, request, response, filterChain);
}
```

3）从容器中获取Filter的delegate实例

容器会搜索springSecurityFilterChain实例，并返回该代理。

```
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
    //从WebApplicationContext容器中获取springSecurityFilterChain实例
    //作为filter类的执行代理类，即拦截请求的处理会交由该类来执行
    Filter delegate = wac.getBean(getTargetBeanName(), Filter.class);
    if (isTargetFilterLifecycle()) {
        delegate.init(getFilterConfig());
    }
    return delegate;
}
```

4）springSecurityFilterChain是何时注入到spring容器中的

重点查看HttpSecurityBeanDefinitionParser类的parse方法

在spring解析http标签时使用HttpSecurityBeanDefinitionParser类进行解析，并会调用registerFilterChainProxyIfNecessary方法，该方法会将springSecurityFilterChain注册到容器中。

HttpSecurityBeanDefinitionParser类的createFilterChain方法创建过滤器链。

```
private BeanReference createFilterChain(Element element, ParserContext pc) {
    ...

    //认证提供者集合
    ManagedList<BeanReference> authenticationProviders = new ManagedList<BeanReference>();
    //认证管理类，解析http标签的authentication-manager-ref属性
    BeanReference authenticationManager = createAuthenticationManager(element, pc,authenticationProviders);

    boolean forceAutoConfig = isDefaultHttpConfig(element);
    //http配置信息，配置请求信息
    //创建CsrfFilter跨域攻击，
    //SecurityContextPersistenceFilter，保存更新一个securityContext对象
    //创建sessionManagerFilter，管理session相关信息，并发访问，认证信息
    //WebAsyncManagerIntegrationFilter,会把SecurityContext设置到异步线程中
    //RequestCacheFilter，请求的缓存处理
    //...
    HttpConfigurationBuilder httpBldr = new HttpConfigurationBuilder(element,
            forceAutoConfig, pc, portMapper, portResolver, authenticationManager);

    //http配置信息，配置认证信息，
    //在配置信息时会需要http相关的过滤设置，因此会在方法中传入
    //AnonymousAuthenticationFilter，匿名认证访问过滤
    //RememberMeAuthenticationProvider，系统记住我过滤器
    //...
    AuthenticationConfigBuilder authBldr = new AuthenticationConfigBuilder(element,
            forceAutoConfig, pc, httpBldr.getSessionCreationPolicy(),
            httpBldr.getRequestCache(), authenticationManager,
            httpBldr.getSessionStrategy(), portMapper, portResolver,
            httpBldr.getCsrfLogoutHandler());

    //在http配置中设置授权相关的过滤
    httpBldr.setLogoutHandlers(authBldr.getLogoutHandlers());
    httpBldr.setEntryPoint(authBldr.getEntryPointBean());
    httpBldr.setAccessDeniedHandler(authBldr.getAccessDeniedHandlerBean());

    authenticationProviders.addAll(authBldr.getProviders());

    //过滤器集合
    List<OrderDecorator> unorderedFilterChain = new ArrayList<OrderDecorator>();

    unorderedFilterChain.addAll(httpBldr.getFilters());
    unorderedFilterChain.addAll(authBldr.getFilters());
    unorderedFilterChain.addAll(buildCustomFilterList(element, pc));

    //排序过滤器集合
    Collections.sort(unorderedFilterChain, new OrderComparator());
    checkFilterChainOrder(unorderedFilterChain, pc, pc.extractSource(element));

    // The list of filter beans
    List<BeanMetadataElement> filterChain = new ManagedList<BeanMetadataElement>();

    for (OrderDecorator od : unorderedFilterChain) {
        filterChain.add(od.bean);
    }

    return createSecurityFilterChainBean(element, pc, filterChain);
}
```

### http标签的auto-config功能

如果配置的auto-config属性为true，则会自动注入登录表单、基础认证、登出url，登出的处理类。

1）标签的处理类

在BeanDefinitionParserDelegate类的parseCustomElement方法会根据元素标签的命名空间获取处理类，然后进行处理。http标签的handler为SecurityNamespaceHandler。

在XmlReaderContext实例创建时，会设置NamespaceHandlerResolver，而该接口的实现类为DefaultNamespaceHandlerResolver。DefaultNamespaceHandlerResolver的实例化时，会使用classLoader加载META-INF/spring.handlers文件，这些文件指定了相关namespace的处理类。在spring-beans，spring-security-config包中都存在文件spring.handlers，文件中定义了相关的处理类。

spring-security-config的jar包中处理类的定义信息

```
http\://www.springframework.org/schema/security=org.springframework.security.config.SecurityNamespaceHandler
```

2）SecurityNamespaceHandler标签解析

```
//该类中存在一个私有变量，定义标签解析的map集合，用于匹配相应标签的解析类
private final Map<String, BeanDefinitionParser> parsers = 
    new HashMap<String, BeanDefinitionParser>();

//init初始化方法加载解析器
private void loadParsers() {
    ...
    //判断类加载器中是否加载了FilterChainProxy类，如果加载了
    if(ClassUtils.isPresent(FILTER_CHAIN_PROXY_CLASSNAME, 
        getClass().getClassLoader())) {
        //http标签的处理类
        parsers.put(Elements.HTTP, new HttpSecurityBeanDefinitionParser());
        ...
    }
}
```

3）HttpSecurityBeanDefinitionParser的auto-config属性的处理

在解析http标签时，AuthenticationConfigBuilder类中定义了ATT_AUTO_CONFIG成员变量，如果配置的auto-config属性为true，则会自动注入登录表单、基础认证、登出url，登出的处理类。

### intercept-url属性的处理

1）解析intercept-url标签

在HttpConfigurationBuilder类的createFilterSecurityInterceptor方法中调用FilterInvocationSecurityMetadataSourceParser.createSecurityMetadataSource访问完成标签的解析工作。

默认使用DefaultFilterInvocationSecurityMetadataSource（如果http没有设置use-expressions属性），并且关联了intercept-url设置的属性信息（access，pattern等属性）。

2）请求拦截器链的执行

每个请求都最终会被拦截并交由filterChains链来处理，而这些filterChains被封装在了FilterChainProxy对象中。将断点定位到FilterChainProxy类的doFilter方法上，查看执行。

3）权限拦截与认证

FilterSecurityInterceptor类invoke方法会调用父类的beforeInvocation方法（AbstractSecurityInterceptor类）。

### authentication-manager标签的功能