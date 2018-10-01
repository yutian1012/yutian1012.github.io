---
title: spring中BeanWrapper对象的理解
tags: [spring]
---

参考：https://blog.csdn.net/zhiweianran/article/details/7919129

1）BeanWrapper的介绍

BeanWrapper大部分情况下是在spring-ioc内部进行使用，通过BeanWrapper，spring-ioc容器可以用统一的方式来访问bean的属性。用户很少需要直接使用BeanWrapper进行编程。

BeanWrapper是对Bean的包装，其接口中所定义的功能很简单包括设置获取被包装的对象，获取被包装bean的属性描述器，由于BeanWrapper接口是PropertyAccessor的子接口，因此其也可以设置以及访问被包装对象的属性值。

2）BeanWrapperImpl类

BeanWrapperImpl类是对BeanWrapper接口的默认实现，它包装了一个bean对象，缓存了bean的内省结果，并可以访问bean的属性、设置bean的属性值。

BeanWrapperImpl类的内部结构信息

```
/** The wrapped object */
private Object object;//封装的实体对象

private String nestedPath = "";//嵌套路径

private Object rootObject;//根对象

private AccessControlContext acc;

/**
 * Cached introspections results for this object, to prevent encountering
 * the cost of JavaBeans introspection every time.
   缓存类的内省信息
 */
private CachedIntrospectionResults cachedIntrospectionResults;

/**
 * Map with cached nested BeanWrappers: nested path -> BeanWrapper instance.
 */
private Map<String, BeanWrapperImpl> nestedBeanWrappers;

private boolean autoGrowNestedPaths = false;

private int autoGrowCollectionLimit = Integer.MAX_VALUE;
```

3）查看BeanWrapperImpl的使用

AbstractAutowireCapableBeanFactory类的instantiateBean方法返回BeanWrapper实例，内部使用BeanWrapperImpl作为其实现类。

```
beanInstance=...//实例化对象后
//使用BeanWrapperImpl包装对象
BeanWrapper bw = new BeanWrapperImpl(beanInstance);
initBeanWrapper(bw);
return bw;
```

查看构造函数

```
public BeanWrapperImpl(Object object) {
    registerDefaultEditors();//内部设置属性defaultEditorsActive值为true
    setWrappedInstance(object);
}
```