---
title: spring容器的DefaultListableBeanFactory初始化过程
tags: [spring]
---

参考：https://www.cnblogs.com/davidwang456/p/4187012.html

DefaultListableBeanFactory类是真正第一个可以独立的IOC容器，而且后面的ApplicationContext据说也是依据它来建立。DefaultListableBeanFactory既可以作为一个单独的beanFactory，也可以作为自定义beanFactory的父类。

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

AbstractAutowireCapableBeanFactory父类构造

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

DefaultListableBeanFactory采用大量的集合变量来保存一些涉及到的数据。

```
private static Class<?> javaxInjectProviderClass = null;

static {
    try {
        //返回了javax.inject.Provider的一个实例
        javaxInjectProviderClass =
                ClassUtils.forName("javax.inject.Provider", DefaultListableBeanFactory.class.getClassLoader());
    }
    catch (ClassNotFoundException ex) {
        // JSR-330 API not available - Provider interface simply not supported then.
    }
}


//系统的DefaultListableBeanFactory容器集合
private static final Map<String, Reference<DefaultListableBeanFactory>> serializableFactories =new ConcurrentHashMap<String, Reference<DefaultListableBeanFactory>>(8);

/** Optional id for this factory, for serialization purposes */
private String serializationId;

/** Whether to allow re-registration of a different definition with the same name */
private boolean allowBeanDefinitionOverriding = true;

/** Whether to allow eager class loading even for lazy-init beans */
private boolean allowEagerClassLoading = true;

/** Optional OrderComparator for dependency Lists and arrays */
//对依赖集合进行排序
private Comparator<Object> dependencyComparator;

/** Resolver to use for checking if a bean definition is an autowire candidate */
//处理bean定义信息是否为自动注入的候选对象
private AutowireCandidateResolver autowireCandidateResolver = new SimpleAutowireCandidateResolver();

/** Map from dependency type to corresponding autowired value */
//处理的类型依赖对象集合
private final Map<Class<?>, Object> resolvableDependencies = new HashMap<Class<?>, Object>(16);

/** Map of bean definition objects, keyed by bean name */
//BeanDefinition定义的集合
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(64);

/** Map of singleton and non-singleton bean names keyed by dependency type */
//类型对应的BeanNames数组集合
private final Map<Class<?>, String[]> allBeanNamesByType = new ConcurrentHashMap<Class<?>, String[]>(64);

/** Map of singleton-only bean names keyed by dependency type */
private final Map<Class<?>, String[]> singletonBeanNamesByType = new ConcurrentHashMap<Class<?>, String[]>(64);

/** List of bean definition names, in registration order */
//所有的BeanDefinition的name集合
private final List<String> beanDefinitionNames = new ArrayList<String>();

/** Whether bean definition metadata may be cached for all beans */
//beanDefinition的元数据是否允许修改，与缓存设置相关
private boolean configurationFrozen = false;

/** Cached array of bean definition names in case of frozen configuration */
//缓存的beanDefinition的name集合
private String[] frozenBeanDefinitionNames;
```