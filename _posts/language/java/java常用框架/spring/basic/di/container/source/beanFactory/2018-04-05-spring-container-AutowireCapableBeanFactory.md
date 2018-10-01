---
title: spring容器的AutowireCapableBeanFactory接口
tags: [spring]
---

1）介绍（查看类的英文注释）

可以使用这个接口集成其它框架，捆绑并填充并不由Spring管理生命周期并已存在的实例。像集成WebWork的Actions和TapestryPage就很实用。

一般应用开发者不会使用这个接口，所以像ApplicationContext这样的外观实现类不会实现这个接口，可以通过ApplicationContext的getAutowireCapableBeanFactory接口获取。

接口定义了5种自动装配策略：不注入AUTOWIRE_NO，使用beanname策略装配AUTOWIRE_BY_NAME，使用类型装配策略AUTOWIRE_BY_TYPE，使用构造器装配策略AUTOWIRE_CONSTRUCTOR，自动装配策略AUTOWIRE_AUTODETECT。

2）接口方法

AutowireCapableBeanFactory接口定义了自动装配bean对象时常用的API。

```
public interface AutowireCapableBeanFactory extends BeanFactory {

    //提供了5种自动装配策略
    //策略1：不注入
    int AUTOWIRE_NO = 0;

    //策略2：使用bean name策略装配
    int AUTOWIRE_BY_NAME = 1;

    //策略3：使用类型装配策略
    int AUTOWIRE_BY_TYPE = 2;

    //策略4：使用构造器装配策略
    int AUTOWIRE_CONSTRUCTOR = 3;

    //策略5：自动装配策略
    @Deprecated
    int AUTOWIRE_AUTODETECT = 4;

    //-------------------------------------------------------------------------
    // Typical methods for creating and populating external bean instances
    //创建和填充外部bean实例
    //-------------------------------------------------------------------------
    
    <T> T createBean(Class<T> beanClass) throws BeansException;

    //使用autowire BeanProperties装配属性
    void autowireBean(Object existingBean) throws BeansException;

    //自动装配属性,填充属性值,使用诸如setBeanName,setBeanFactory这样的工厂回调填充属性,最后还要调用post processor
    Object configureBean(Object existingBean, String beanName) throws BeansException;

    Object resolveDependency(DependencyDescriptor descriptor, String beanName) throws BeansException;

    //-------------------------------------------------------------------------
    // Specialized methods for fine-grained control over the bean lifecycle
    //在bean的生命周期进行细粒度控制的专门方法
    //-------------------------------------------------------------------------

    //指定创建类时使用的自动装配策略
    Object createBean(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;

    //根据给定装配策略的bean的class自动装配对象
    Object autowire(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;

    void autowireBeanProperties(Object existingBean, int autowireMode, boolean dependencyCheck)
            throws BeansException;

    void applyBeanPropertyValues(Object existingBean, String beanName) throws BeansException;

    Object initializeBean(Object existingBean, String beanName) throws BeansException;

    //bean初始化前调用BeanPostProcessor，进一步初始化对象
    Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)throws BeansException;

    //bean初始化后调用BeanPostProcessor，进一步初始化对象
    Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException;

    //销毁bean对象
    void destroyBean(Object existingBean);

    Object resolveDependency(DependencyDescriptor descriptor, String beanName,
            Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException;
}
```

注：该接口与bean声明周期的实例化相关。