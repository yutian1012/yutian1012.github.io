---
title: spring容器注册配置类
tags: [spring]
---

AnnotationConfigApplicationContext实现了BeanDefinitionRegistry接口。因此，这里的register对象，也可以理解为上下文本身。

解析配置类信息，scope作用域（scopeMetadataResolver），条件化bean（conditionEvaluator，用于profile等特性），beanname的处理（beanNameGenerator）。将配置bean信息存储到definitionHolder对象中。然后将上面用到的所有信息都设置到BeanDefinitionRegistry（该类有一个实现DefaultListableBeanFactory）对象中。

1）register方法

```
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
    this();
    //注册配置类信息
    register(annotatedClasses);
    //根据配置信息加载相关的bean定义到容器中
    refresh();
}
```

2）处理过程

```
//接收一个不定参数列表，即配置类数组
public void register(Class<?>... annotatedClasses) {
    ...
    this.reader.register(annotatedClasses);
}
```

register方法将配置类交由AnnotatedBeanDefinitionReader来处理（即这里的this.reader成员变量）。具体的register方法会变量该配置类数组，并逐个加载配置类。

AnnotatedBeanDefinitionReader类的register方法

```
public void register(Class<?>... annotatedClasses) {
    for (Class<?> annotatedClass : annotatedClasses) {
        registerBean(annotatedClass);
    }
}
```

3）AnnotatedBeanDefinitionReader类

AnnotatedBeanDefinitionReader类成员变量

```
//这里的register即AnnotationConfigApplicationContext，因为该上下文实现了BeanDefinitionRegistry接口。
private final BeanDefinitionRegistry registry;

//spring容器解析bean，生成bean对应的name生成器
private BeanNameGenerator beanNameGenerator = new AnnotationBeanNameGenerator();

//解析容器bean的生命周期处理类
private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();

//计算条件，用于bean的条件实例化处理
private ConditionEvaluator conditionEvaluator;
```

3）具体处理单个配置类，查看registerBean方法

该方法主要是解析配置类，并将关键信息提取并设置到beanDefinition对象中，注册到BeanDefinitionRegistry对象中

```
//这里的annotatedClass是spring配置类
public void registerBean(Class<?> annotatedClass, String name,
        @SuppressWarnings("unchecked") Class<? extends Annotation>... qualifiers) {
    
    //bean定义的注解通用信息，包括传递类的注解结果化信息，包括类上的注解信息，注解的属性，注解的方法等元数据信息，通过该类可以方便获取
    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
    //根据条件表达式判断是否忽略对该配置类的解析
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
        return;
    }

    //从bean定义的注解信息对象中获取定义的生命周期信息scope
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
    abd.setScope(scopeMetadata.getScopeName());
    //生成beanname信息
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
    //解析注解的metadata信息，直接设置到AnnotatedGenericBeanDefinition对象中，方便获取注解信息
    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
    //处理qualified标识符，设置到AnnotatedGenericBeanDefinition对象中
    if (qualifiers != null) {
        for (Class<? extends Annotation> qualifier : qualifiers) {
            if (Primary.class.equals(qualifier)) {
                abd.setPrimary(true);
            }
            else if (Lazy.class.equals(qualifier)) {
                abd.setLazyInit(true);
            }
            else {
                abd.addQualifier(new AutowireCandidateQualifier(qualifier));
            }
        }
    }
    
    //该类包装了AnnotatedGenericBeanDefinition的bean定义信息
    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    //判断并应用代理，如proxyTargetClass
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    //将bean的定义信息注册到register对象中，即能够在应用上下文中能够获取
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```