---
title: spring容器的ApplicationContext与BeanFactory的关系
tags: [spring]
---

spring的ApplicationContext包含了beanFactory的所有功能，还有其他额外功能。

1）ApplicationContext的定义信息

```
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
        MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    ...
}
```

从定义信息可以看到该接口继承了ListableBeanFactory，HierarchicalBeanFactory等BeanFactory接口。因此，该接口比能够提供BeanFactory接口定义的功能。

2）BeanFactory接口

```
public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";

    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException;

    boolean containsBean(String name);

    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;

    Class<?> getType(String name) throws NoSuchBeanDefinitionException;

    String[] getAliases(String name);
}
```

注：BeanFactory接口中定义了bean对象的获取方式，类似于bean工厂产生bean对象。

3）一个实现AbstractApplicationContext

AbstractApplicationContext是一个抽象类，该类中对BeanFactory提供的接口进行了部分实现。

a.getBean方法的实现

```
@Override
public <T> T getBean(Class<T> requiredType) throws BeansException {
    assertBeanFactoryActive();
    return getBeanFactory().getBean(requiredType);
}
```

其中getBeanFactory方法是AbstractApplicationContext类中定义的模板方法。

4）GenericApplicationContext类

GenericApplicationContext类是AbstractApplicationContext的一个实现类，该类中对AbstractApplicationContext类中定义的模板方法进行了实现。

```
//成员变量
private final DefaultListableBeanFactory beanFactory;

@Override
public final ConfigurableListableBeanFactory getBeanFactory() {
    //返回成员变量
    return this.beanFactory;
}
```

那么成员变量beanFactory是何时初始化的呢？GenericApplicationContext类的默认构造函数被调用时就会初始化beanFactory对象。

```
public GenericApplicationContext() {
    this.beanFactory = new DefaultListableBeanFactory();
}
```

注：综上，可以看出ApplicationContext是基于BeanFactory构建的。