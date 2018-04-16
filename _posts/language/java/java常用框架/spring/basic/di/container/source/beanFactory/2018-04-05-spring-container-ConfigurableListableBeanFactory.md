---
title: spring容器的ConfigurableListableBeanFactory
tags: [spring]
---

ConfigurableListableBeanFactory是一个集成接口，提供解析、修改bean定义的设置。

其中ListableBeanFactory用于提供容器内bean实例的枚举功能。AutowireCapableBeanFactory提供容器的自动配置功能，ConfigurableBeanFactory接口提供了定义BeanFactory的配置，如类加载器，类型转化，属性编辑器等。

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