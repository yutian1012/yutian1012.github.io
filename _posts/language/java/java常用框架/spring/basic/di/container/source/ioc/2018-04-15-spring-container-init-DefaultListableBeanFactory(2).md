---
title: spring容器的DefaultListableBeanFactory初始化过程
tags: [spring]
---

参考：https://www.cnblogs.com/davidwang456/p/4187012.html

DefaultListableBeanFactory类是真正第一个可以独立的IOC容器，而且后面的ApplicationContext据说也是依据它来建立。

DefaultListableBeanFactory既可以作为一个单独的beanFactory，也可以作为自定义beanFactory的父类。即自定义BeanFactory类使可以继承该类，并实现自定义的一些方法。

1）DefaultListableBeanFactory初始化

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

2）DefaultListableBeanFactory的成员变量

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