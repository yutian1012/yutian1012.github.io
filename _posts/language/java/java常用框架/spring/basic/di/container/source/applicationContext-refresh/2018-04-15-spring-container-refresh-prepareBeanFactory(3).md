---
title: spring容器对BeanFactory对象进行填充
tags: [spring]
---

AbstractApplicationContext类的prepareBeanFactory方法，对BeanFactory进行初始化配置，包括设置ClassLoader以及post-processors，添加应用的回调处理，设置环境对象等信息。此时BeanFactory就追加了一些默认的功能。

参考:https://www.cnblogs.com/tangpu/p/3203895.html

```
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    //设置classLoader（类加载器）
    beanFactory.setBeanClassLoader(getClassLoader());
    //设置bean对象的SpEL表达式的解析器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver());
    //为beanFactory增加了一个默认的propertyEditor属性编辑器，这里主要设置资源类型（IO相关的资源信息）的编辑器
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // Configure the bean factory with context callbacks.
    //配置应用上下文的回调，一般回调都是使用Aware来命名的，
    //aware接口是用来给bean注入一些资源的接口。
    /*ApplicationContextAwareProcessor类对继承自ApplicationContextAware的bean进行处理，调用其setApplicationContext。从而使普通bean可以通过实现ApplicationContextAware得到ApplicationContext。*/
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    /*忽略给定接口的自动装配功能，典型应用是通过其他方式就诶下Application上下文注册以来，类似于BeanFactory通过BeanFactoryAware进行注入，或者ApplicatinContext通过ApplicationContextAware进行注入*/
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);

    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    /*设置了几个自动装配的特殊规则，如果是BeanFactory类型，则注入beanFactory对象，如果是ResourceLoader、ApplicationEventPublisher、ApplicationContext类型则注入当前对象（applicationContext对象）。*/
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    //spring的loadTimeWeaver主要是通过instrumentation的动态字节码增强在装载期注入依赖。
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // Set a temporary ClassLoader for type matching.
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // Register default environment beans.
    //注册环境类型对象，先判断是否配置了，如果没有配置则会自动设置
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```