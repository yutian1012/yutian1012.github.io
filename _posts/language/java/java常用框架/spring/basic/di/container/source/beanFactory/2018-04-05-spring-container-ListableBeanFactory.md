---
title: spring容器的ListableBeanFactory接口
tags: [spring]
---

ListableBeanFactory接口提供容器内bean实例的枚举功能，如按照注解获取对象，获取BeanDefinition的名称数组，以及根据类型获取实例对象等。

```
public interface ListableBeanFactory extends BeanFactory {

    //查看是否包含BeanDefinition定义信息
    boolean containsBeanDefinition(String beanName);

    //获取BeanDefinition的定义量
    int getBeanDefinitionCount();

    String[] getBeanDefinitionNames();

    String[] getBeanNamesForType(Class<?> type);
    
    //includeNonSingletons是否包括非单例对象
    String[] getBeanNamesForType(Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);

    <T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException;
    
    <T> Map<String, T> getBeansOfType(Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
            throws BeansException;
    //根据Annotation注解信息获取beannames的数组
    String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);

    Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;

    <A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType) throws NoSuchBeanDefinitionException;
}
```