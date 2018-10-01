---
title: spring容器的ConfigurableListableBeanFactory集成接口
tags: [spring]
---

1）介绍（查看类的英文注释）

提供beandefinition的解析，注册功能，再对单例来个预加载(解决循环依赖问题)。

该类是一个重要的类，ConfigurableListableBeanFactory是一个集成接口，提供解析、修改bean定义的设置。实现了ListableBeanFactory,AutowireCapableBeanFactory,ConfigurableBeanFactory接口，即综合了很多的功能。

其中ListableBeanFactory用于提供容器内bean实例的枚举功能。AutowireCapableBeanFactory提供容器的自动配置功能，ConfigurableBeanFactory接口提供了定义BeanFactory的配置，如类加载器，类型转化，属性编辑器等。

注：具体使用的容器大多也都是以该接口作为基础接口进一步实现的。

2）接口方法

```
public interface ConfigurableListableBeanFactory
        extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {

    //忽略给定类型的依赖自动装配，如String类型的变量装配默认为null
    void ignoreDependencyType(Class<?> type);

    //忽略指定的接口类型的自动装配，如applicationContexts的自动装配忽略掉，而是使用ApplicationContextAware的方式对bean进行applicationContexts的装配
    void ignoreDependencyInterface(Class<?> ifc);

    //针对特定的依赖类型，自动装配指定对象
    void registerResolvableDependency(Class<?> dependencyType, Object autowiredValue);

    //判断给定的beanname是否是特定依赖（特定依赖对象通过DependencyDescriptor对象描述来指定）自动装配的候选对象
    boolean isAutowireCandidate(String beanName, DependencyDescriptor descriptor) throws NoSuchBeanDefinitionException;

    //获取beanname的BeanDefiniton对象定义
    BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

    //冻结BeanDefinition信息，即不允许再对BeanDefinition定义信息进行修改
    void freezeConfiguration();

    boolean isConfigurationFrozen();

    //处理非懒加载的单例对象的实例化
    void preInstantiateSingletons() throws BeansException;
}
```