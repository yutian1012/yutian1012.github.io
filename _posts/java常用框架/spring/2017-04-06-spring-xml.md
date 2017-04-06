---
title: spring 中常用的配置标签
tags: [spring]
---

### 1. annotation-config的作用

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