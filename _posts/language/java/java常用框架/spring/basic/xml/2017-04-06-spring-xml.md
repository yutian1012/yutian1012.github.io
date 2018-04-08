---
title: spring 中常用的配置标签
tags: [spring]
---

### 1. annotation-config的作用（spring mvc）

使用@Autowired注解，必须事先在Spring容器中声明AutowiredAnnotationBeanPostProcessor的Bean：

```
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor "/>
```

使用 @Required注解，就必须声明RequiredAnnotationBeanPostProcessor的Bean：

```
<bean class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/>
```

类似地，使用@Resource、@PostConstruct、@PreDestroy等注解就必须声明 CommonAnnotationBeanPostProcessor；使用@PersistenceContext注解，就必须声明 PersistenceAnnotationBeanPostProcessor的Bean。使用annotation-config可以简化配置。

注：context:annotation-config隐式地向 Spring容器注册AutowiredAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor以及PersistenceAnnotationBeanPostProcessor这4个BeanPostProcessor。

另外：

```
<context:component-scan base-package="pack.pack"/>
```

注：component-scan也包含了自动注入上述的processor。

参考：http://www.cnblogs.com/iuranus/archive/2012/07/19/2599084.html

### 2. annotation-driven的作用（spring mvc）

会自动注册DefaultAnnotationHandlerMapping 与AnnotationMethodHandlerAdapter 这两个bean

1）spring 3.1 later：

Spring3.0.x中使用了annotation-driven后，缺省使用DefaultAnnotationHandlerMapping来注册handler method和request的mapping关系。 

AnnotationMethodHandlerAdapter来在实际调用handlermethod前对其参数进行处理。

在dispatcherServlet中，当用户未注册自定义的ExceptionResolver时，注册AnnotationMethodHandlerExceptionResolver来对使用@ExceptionHandler标注的异常处理函数进行解析处理(这也导致当用户注册了自定义的exeptionResolver时将可能导致无法处理@ExceptionHandler)。

2）在spring mvc 3.1中，对应变更为

DefaultAnnotationHandlerMapping -> org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping

AnnotationMethodHandlerAdapter -> org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter

AnnotationMethodHandlerExceptionResolver -> ExceptionHandlerExceptionResolver

注：以上都在使用了annotation-driven后自动注册。 

注2：对应分别提供了AbstractHandlerMethodMapping,AbstractHandlerMethodAdapter和AbstractHandlerMethodExceptionResolver以便于让用户更方便的实现自定义的实现类。

3）可选配置

```
<mvc:annotation-driven  message-codes-resolver="bean ref" validator="" conversion-service="">
   
    <mvc:return-value-handlers>
        <bean></bean>
    </mvc:return-value-handlers>
    
    <mvc:argument-resolvers>
    </mvc:argument-resolvers>
    
    <mvc:message-converters>
    </mvc:message-converters>
</mvc:annotation-driven>
```