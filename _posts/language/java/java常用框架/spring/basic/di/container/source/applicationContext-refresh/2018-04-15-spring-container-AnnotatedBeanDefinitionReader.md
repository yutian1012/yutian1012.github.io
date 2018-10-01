---
title: spring中AnnotatedBeanDefinitionReader对象
tags: [spring]
---

该对象的初始化，不仅仅是针对自身的初始化，还包括向BeanFactory中加入postProcessor对象，可以在BeanFactory的beanDefinitionMap对象中查看。这些注入的后置处理器将会在refresh的第五步invokeBeanFactoryPostProcessors中被调用执行。

1）AnnotatedBeanDefinitionReader对象的初始化

AnnotationConfigApplicationContext类初始化时，会生成AnnotatedBeanDefinitionReader对象实例。

```
public AnnotationConfigApplicationContext() {
    //构建AnnotatedBeanDefinitionReader对象时，将应用上下文实例传入
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

2）AnnotatedBeanDefinitionReader构造函数

```
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
    this(registry, getOrCreateEnvironment(registry));
}
```

注：getOrCreateEnvironment方法获取Environment对象，如果registry是ApplicationContext类，则会返回上下文中的环境对象，因为ApplicationContext继承了EnvironmentCapable接口，否则创建一个StandardEnvironment对象返回。这里registry是AnnotationConfigApplicationContext，即是一个应用上下文，因此会返回上下文中的环境对象。

调用构造函数，传入获取的Environment对象

```
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    ...
    this.registry = registry;//设置registry属性为当前应用上下文
    //初始化条件求值对象
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    //向register（应用上下文）中设置配置处理器
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

3）配置后置处理器

查看AnnotationConfigUtils.registerAnnotationConfigProcessors方法

```
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
            BeanDefinitionRegistry registry, Object source) {
    /*获取BeanFactory对象，如果register是DefaultListableBeanFactory的实例，则直接返回，如果register是GenericApplicationContext，则调用getDefaultListableBeanFactory方法获取，在GenericApplicationContext中定义了DefaultListableBeanFactory成员变量。*/
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        //为BeanFactory设置比较器，处理Order注解，注入时有时会判断Order的顺序
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }
        //注入自动装配候选bean对象的处理器
        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
    }

    //设置4个BeanDefinitionHolder对象
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<BeanDefinitionHolder>(4);

    /*配置ConfigurationClassPostProcessor类，针对配置类的后置处理器，并且通过registerPostProcessor方法注册到上下文中，会注入到BeanFactory的beanDefinitionMap对象中（该对象是一个ConcurrentMap对象）*/
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //配置AutowiredAnnotationBeanPostProcessor后置处理器
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //配置RequiredAnnotationBeanPostProcessor后置处理器
    if (!registry.containsBeanDefinition(REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(RequiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
    //是否使用基于JSR-250注解，如果支持则加入CommonAnnotationBeanPostProcessor处理器
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
    //是否支持JPA，如果支持则加入PersistenceAnnotationBeanPostProcessor处理器
    if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition();
        try {
            def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
                    AnnotationConfigUtils.class.getClassLoader()));
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                    "Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
        }
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    return beanDefs;
}
```