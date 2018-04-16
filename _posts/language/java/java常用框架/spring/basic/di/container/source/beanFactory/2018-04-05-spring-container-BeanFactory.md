---
title: spring容器的BeanFactory
tags: [spring]
---

参考：https://www.cnblogs.com/leftthen/p/5261837.html

1）BeanFactory接口

BeanFactory是Spring-bean容器的根接口。提供获取bean，是否包含bean，是否单例与原型，获取bean类型，bean别名的api。

```
public interface BeanFactory {
    //FactoryBean类型的bean使用&前缀
    String FACTORY_BEAN_PREFIX = "&";

    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException;
    //判断是否包含该bean
    boolean containsBean(String name);
    //是否单例
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    //是否原型方式创建bean
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    //类型是否匹配
    boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;
    //
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    //别名数组
    String[] getAliases(String name);
}
```

注：BeanFactory接口中定义了bean对象的获取方式，类似于bean工厂产生bean对象。

2）一个实现分支

![](/images/spring/core/source/applicantContext/BeanFactory.png)

查看该分支下的实现情况

```
BeanFactory-->ListableBeanFactory-->ConfigurableListableBeanFactory-->DefaultListableBeanFactory
```

3）ListableBeanFactory接口

该接口提供容器内bean实例的枚举功能，如按照注解获取对象，获取BeanDefinition的名称数组，以及根据类型获取实例对象等。

4）ConfigurableListableBeanFactory接口

该接口是一个集成接口，继承了ListableBeanFactory、AutowireCapableBeanFactory和ConfigurableBeanFactory接口。提供解析、修改bean定义，并与初始化单例。

```
public interface ConfigurableListableBeanFactory
        extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {
    ...
}
```

5）AutowireCapableBeanFactory接口

该接口提供了自动装配bean对象时常用的API。

6）ConfigurableBeanFactory接口

定义BeanFactory的配置，如类加载器，类型转化，属性编辑器，,BeanPostProcessor，作用域，bean定义，处理bean依赖关系，合并其他ConfigurableBeanFactory，bean如何销毁等。

7）DefaultListableBeanFactory实现类

该类实现了ConfigurableListableBeanFactory接口，因此该类能够提供ConfigurableListableBeanFactory接口定义的方法集合。

