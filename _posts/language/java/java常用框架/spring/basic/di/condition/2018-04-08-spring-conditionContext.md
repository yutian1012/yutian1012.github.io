---
title: spring条件上下文对象
tags: [spring]
---

1）Condition条件化接口

```
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

该接口提供了两个对象参数ConditionContext和AnnotatedTypeMetadata。

2）ConditionConext接口

通过该接口可以获取容器中的bean定义信息，bean是否存在，容器的环境信息，以及资源信息。

```
public interface ConditionContext {
    //借助getRegistry()返回的BeanDefinitionRegistry检查bean定义
    BeanDefinitionRegistry getRegistry();
    
    //借助getBeanFactory()返回的ConfigurableListableBeanFactory检查bean是否存在，甚至探查bean的属性
    ConfigurableListableBeanFactory getBeanFactory();
    
    //借助getEnvironment()返回的Environment检查环境变量是否存在以及它的值是什么
    Environment getEnvironment();

    //读取并探查getResourceLoader()返回的ResourceLoader所加载的资源
    ResourceLoader getResourceLoader();
    
    //借助getClassLoader()返回的ClassLoader加载并检查类是否存在
    ClassLoader getClassLoader();
}
```

注：BeanDefinitionRegistry，该类的作用主要是向注册表中注册BeanDefinition实例，完成注册的过程。因此BeanDefinitionRegistry对象能够获取bean对的定义信息。

3）AnnotatedTypeMetadata接口

AnnotatedTypeMetadata能够让我们检查带有@Bean注解的方法上还有什么其他的注解。

```
public interface AnnotatedTypeMetadata {

    boolean isAnnotated(String annotationType);

    Map<String, Object> getAnnotationAttributes(String annotationType);

    Map<String, Object> getAnnotationAttributes(String annotationType, boolean classValuesAsString);

    MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationType);

    MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationType, boolean classValuesAsString);
}
```

借助isAnnotated()方法，我们能够判断带有@Bean注解的方法是不是还有其他特定的注解。借助其他的那些方法，我们能够检查@Bean注解的方法上其他注解的属性。