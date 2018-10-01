---
title: spring容器的ListableBeanFactory接口
tags: [spring]
---

1）介绍（查看类的英文注释）

获取bean时，Spring鼓励使用这个接口定义的api，还有个Beanfactory方便使用。其他的4个接口都是不鼓励使用的。

提供容器中bean迭代的功能，不再需要一个个bean地查找。比如可以一次获取全部的bean(太暴力了)，根据类型获取bean。在看SpringMVC时，扫描包路径下的具体实现策略就是使用的这种方式(那边使用的是BeanFactoryUtils封装的api)。

如果同时实现了HierarchicalBeanFactory，返回值不会考虑父类BeanFactory，只考虑当前factory定义的类。当然也可以使用BeanFactoryUtils辅助类来查找祖先工厂中的类。

这个接口中的方法只会考虑本factory定义的bean。这些方法会忽略ConfigurableBeanFactory的registerSingleton注册的单例bean（getBeanNamesOfType和getBeansOfType是例外，一样会考虑手动注册的单例）。当然BeanFactory的getBean一样可以透明访问这些特殊bean。当然在典型情况下，所有的bean都是由external bean定义，所以应用不需要顾虑这些差别。

注意:getBeanDefinitionCount和containsBeanDefinition的实现方法因为效率比较低，还是少用为好。

2）接口方法

ListableBeanFactory接口提供容器内bean实例的枚举功能，如按照注解获取对象，获取BeanDefinition的名称数组，以及根据类型获取实例对象等。

```
public interface ListableBeanFactory extends BeanFactory {

    //查看是否包含BeanDefinition定义信息，忽略父factory和其他factory注册的单例bean
    boolean containsBeanDefinition(String beanName);

    //获取BeanDefinition的定义量
    int getBeanDefinitionCount();
    
    //获取工厂中定义的所有bean的name，一样不考虑父factory和其他factory注册的单例bean
    String[] getBeanDefinitionNames();

    //获取给定类型的beannames（包括子类）。
    //通过bean定义或者FactoryBean的getObjectType判断.
    //注意:这个方法仅检查顶级bean.它不会检查嵌套的bean.
    //FactoryBean创建的bean会匹配为FactoryBean而不是原始类型.
    //一样不会考虑父factory中的bean,可以使用BeanFactoryUtils中的beanNamesForTypeIncludingAncestors.
    //这个版本的getBeanNamesForType会匹配所有类型的bean,包括单例,原型,FactoryBean。在大多数实现中返回结果跟getBeanNamesOfType(type,true,true)一样.
    //返回的bean names会根据backend配置的进行排序.
    String[] getBeanNamesForType(Class<?> type);
    
    //includeNonSingletons是否包括非单例对象
    String[] getBeanNamesForType(Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);

    <T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException;
    
    <T> Map<String, T> getBeansOfType(Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
            throws BeansException;

    //根据Annotation注解信息获取beannames的数组
    String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);

    Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;

    //查找一个类上的注解,如果找不到,父类,接口使用注解也算
    <A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType) throws NoSuchBeanDefinitionException;
}
```

注：通过方法可以看出，该接口中返回的类型都是以集合的形式返回的多值，要么是数组，要么是Map对象，这里的可枚举也是指这个意思，返回的不是单个对象，而是集合类型的多值数据。