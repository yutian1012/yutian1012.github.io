---
title: spring容器的DefaultListableBeanFactory初始化过程
tags: [spring]
---

参考：https://www.cnblogs.com/davidwang456/p/4187012.html

DefaultListableBeanFactory类是真正第一个可以独立的IOC容器，而且后面的ApplicationContext据说也是依据它来建立。

DefaultListableBeanFactory既可以作为一个单独的beanFactory，也可以作为自定义beanFactory的父类。即自定义BeanFactory类使可以继承该类，并实现自定义的一些方法。

1）DefaultListableBeanFactory类的继承层次

默认实现了ListableBeanFactory和BeanDefinitionRegistry接口。

![](/images/spring/core/source/applicantContext/DefaultListableBeanFactory-hirerachy.jpg)

AbstractAutowireCapableBeanFactory实现属性的自动绑定功能。

ConfigurableListableBeanFactory提供了对bean定义的分析和修改的便利方法，同时也提供了对单例的预实例化。

ListableBeanFactory提供枚举所有的bean实例，而不是客户端通过名称一个一个的查询得出所有的实例。

BeanDefinitionRegistry提供了beanDefinition的管理。

2）DefaultListableBeanFactory初始化

```
DefaultListableBeanFactory factory=new DefaultListableBeanFactory();
```

构造方法，通过查看构造可以看出，该类仅仅调用了super（父类的默认构造）

```
public DefaultListableBeanFactory() {
    super();
}
```

调用父类AbstractAutowireCapableBeanFactory构造函数

```
public AbstractAutowireCapableBeanFactory() {
    super();//继续上调默认构造器
    ignoreDependencyInterface(BeanNameAware.class);
    ignoreDependencyInterface(BeanFactoryAware.class);
    ignoreDependencyInterface(BeanClassLoaderAware.class);
}
```

调用IgnoreDependencyInterface方法（自动装配时，忽略的接口类型），所以在自动装配的时候，会自动忽略BeanNameAware.class，BeanFactoryAware.class，BeanClassLoaderAware.class接口类型的类。

3）AbstractAutowireCapableBeanFactory抽象类

AbstractAutowireCapableBeanFactory的作用：提供bean的创建（有construct方法），属性注值, 绑定（包括自动绑定）和初始化。处理运行时bean引用，解析管理的集合，调用初始化方法。

4）DefaultListableBeanFactory的成员变量

DefaultListableBeanFactory采用大量的集合变量来保存解析到的bean定义数据。

```
//系统的DefaultListableBeanFactory容器集合
private static final Map<String, Reference<DefaultListableBeanFactory>> serializableFactories =new ConcurrentHashMap<String, Reference<DefaultListableBeanFactory>>(8);

//处理bean定义信息是否为自动注入的候选对象
private AutowireCandidateResolver autowireCandidateResolver = new SimpleAutowireCandidateResolver();

//处理的类型依赖对象集合
private final Map<Class<?>, Object> resolvableDependencies = new HashMap<Class<?>, Object>(16);

//BeanDefinition定义的集合
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(64);

//所有的BeanDefinition的name集合
private final List<String> beanDefinitionNames = new ArrayList<String>();

...

```

