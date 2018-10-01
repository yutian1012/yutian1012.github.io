---
title: spring容器中根据RootBeanDefinition创建bean实例
tags: [spring]
---

1）调用层次

AbstractBeanFactory类的doGetBean方法，获取RootBeanDefinition对象根据不同的scope使用不同的分支实例化bean。

下面是针对single单例类型变量进行初始化

```
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
        @Override
        public Object getObject() throws BeansException {
            try {
                return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
                ...
            }
        }
    });
    //针对FactoryBean对象的处理
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

注意：上面实例化对象时使用传递匿名内部类方式，最终调用createBean方法。

2）查看createBean方法

createBean方法在AbstractBeanFactory类中是抽象方法，查看其子类的实现。

DefaultListableBeanFactory-->AbstractAutowireCapableBeanFactory-->AbstractBeanFactory，AbstractAutowireCapableBeanFactory继承了AbstractBeanFactory类

AbstractAutowireCapableBeanFactory类中的createBean方法

```
@Override
protected Object createBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
        throws BeanCreationException {

    // Make sure bean class is actually resolved at this point.
    //保证mdb的BeanClass已处理，即后面使用时能够获取BeanClass
    resolveBeanClass(mbd, beanName);

    // Prepare method overrides.
    try {
        //lookup-method和replaced-method标签的配置
        //该方法会从mdb中获取MethodOverrides信息，并设置mo的overloaded属性
        mbd.prepareMethodOverrides();
    }
    catch (BeanDefinitionValidationException ex) {
       ...
    }

    try {
        // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
        //应用容器设置的InstantiationAwareBeanPostProcessors，可自定义实现并返回代理对象（一个扩展点）
        Object bean = resolveBeforeInstantiation(beanName, mbd);
        //如果resolveBeforeInstantiation方法返回对象不为null，则直接返回
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
       ...
    }

    Object beanInstance = doCreateBean(beanName, mbd, args);
   
    return beanInstance;
}
```

3）查看doCreateBean方法（spring定义的生命周期方法）

这是一个关键方法，定义了创建bean对象的整个流程。doCreateBean-->createBeanInstance-->populateBean-->initializeBean，最终返回创建的bean对象。

```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args) {
    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        //从factoryBeanInstanceCache缓存中移除beanname（map集合）
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        //创建bean实例，针对不同的实例化策略采用不同的方式实例化对象
        //其中SimpleInstantiationStrategy实例化策略是利用构造函数的newInstance方式实例化。
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    //从wrapper中获取bean实例
    final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
    //获取bean的class类型
    Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);
    ...
    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        //populateBean方法相当于对bean设置set属性
        populateBean(beanName, mbd, instanceWrapper);
        if (exposedObject != null) {
            //初始化bean对象
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }
    }
    catch (Throwable ex) {
       ...
    }

    ...

    // Register bean as disposable.
    try {
        //向容器中注册对象的销毁方法
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    }
    ...

    return exposedObject;
}
```