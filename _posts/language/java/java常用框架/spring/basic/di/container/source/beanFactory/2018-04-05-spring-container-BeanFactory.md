---
title: spring容器的BeanFactory基础接口
tags: [spring]
---

参考：https://www.cnblogs.com/leftthen/p/5261837.html

参考2：https://www.cnblogs.com/leftthen/p/5259288.html

1）介绍（查看类的英文注释）

每个bean都是通过string类型beanname进行标识。提供了设计模式单例，原型的替代实现。如果beanname配置为单例，应用内只会获取到一个实例。如果配置为原型，那么可以实例化好后填充属性（基于用户的配置）。

注：使用getBean可以代替单例，原型设计模式。

此接口由持有一些bean定义的对象（集合容纳BeanDefinition）来实现，每个bean由String字符串唯一标识。根据bean定义，工厂将返回一个独立对象实例（原型设计模式），或者一个单个共享实例（Singleton设计模式的优雅代替实现，其中该实例是一个factory范围内的单例）。实例的哪种类型将被返回依赖于bean工厂配置：即使API是一样的。从Spring2.0开始，作用域扩展到根据具体的应用上下文，如web环境的request,session。

注：BeanFactory的是应用程序组件注册的中心，同时集中应用程序组件的配置。

Spring的依赖注入功能就是通过实现BeanFactory和其子接口实现的。

BeanFactory作为应用集中配置管理的地方，极大简便应用开发，这样开发人员可以集中与业务。

BeanFactory需要管理bean的生命周期，比如初始化时需要按顺序实现如下接口:

```
 * 1. BeanNameAware's {@code setBeanName}<br>
 * 2. BeanClassLoaderAware's {@code setBeanClassLoader}<br>
 * 3. BeanFactoryAware's {@code setBeanFactory}<br>
 * 4. ResourceLoaderAware's {@code setResourceLoader}
 * (only applicable when running in an application context)<br>
 * 5. ApplicationEventPublisherAware's {@code setApplicationEventPublisher}
 * (only applicable when running in an application context)<br>
 * 6. MessageSourceAware's {@code setMessageSource}
 * (only applicable when running in an application context)<br>
 * 7. ApplicationContextAware's {@code setApplicationContext}
 * (only applicable when running in an application context)<br>
 * 8. ServletContextAware's {@code setServletContext}
 * (only applicable when running in a web application context)<br>
 * 9. {@code postProcessBeforeInitialization} methods of BeanPostProcessors<br>
 * 10. InitializingBean's {@code afterPropertiesSet}<br>
 * 11. a custom init-method definition<br>
 * 12. {@code postProcessAfterInitialization} methods of BeanPostProcessors

 * <p>On shutdown of a bean factory, the following lifecycle methods apply:<br>
 * 1. DisposableBean's {@code destroy}<br>
 * 2. a custom destroy-method definition
```

2）接口方法

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
    //获取指定beanName的类型
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    //别名数组
    String[] getAliases(String name);
}
```

注：BeanFactory接口中定义了bean对象的获取方式，类似于bean工厂产生bean对象。

3）一个实现分支

![](/images/spring/core/source/applicantContext/BeanFactory.png)

查看该分支下的实现情况

```
BeanFactory-->ListableBeanFactory-->ConfigurableListableBeanFactory-->DefaultListableBeanFactory
```

4）ListableBeanFactory接口

该接口提供容器内bean实例的枚举功能，如按照注解获取对象，获取BeanDefinition的名称数组，以及根据类型获取实例对象等。

5）ConfigurableListableBeanFactory接口（重点）

该接口是一个集成接口，继承了ListableBeanFactory、AutowireCapableBeanFactory和ConfigurableBeanFactory接口。提供解析、修改bean定义，并与初始化单例。

```
public interface ConfigurableListableBeanFactory
        extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {
    ...
}
```

6）AutowireCapableBeanFactory接口

该接口提供了自动装配bean对象时常用的API。

7）ConfigurableBeanFactory接口

定义BeanFactory的配置，如类加载器，类型转化，属性编辑器，,BeanPostProcessor，作用域，bean定义，处理bean依赖关系，合并其他ConfigurableBeanFactory，bean如何销毁等。

8）DefaultListableBeanFactory实现类

该类实现了ConfigurableListableBeanFactory接口，因此该类能够提供ConfigurableListableBeanFactory接口定义的方法集合。

