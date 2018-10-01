---
title: spring容器中初始化bean实例--应用factory方法，init-method以及BeanProcessor
tags: [spring]
---

处理BeanNameAware，BeanClassLoaderAware，BeanFactoryAware等接口的后置；调用实现了InitializingBean接口的afterPropertiesSet方法；调用init-method配置的初始化方法；调用BeanPostProcess的postProcessAfterInitialization方法，最后返回对象。

1）查看AbstractAutowireCapableBeanFactory类的initializeBean方法

```
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    ...
    //处理BeanNameAware，BeanClassLoaderAware，BeanFactoryAware等接口的后置
    invokeAwareMethods(beanName, bean);

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        //调用BeanPostProcess的postProcessBeforeInitialization方法
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        //调用实现了InitializingBean接口的afterPropertiesSet方法
        //调用init-method配置的初始化方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                (mbd != null ? mbd.getResourceDescription() : null),
                beanName, "Invocation of init method failed", ex);
    }

    if (mbd == null || !mbd.isSynthetic()) {
        //调用BeanPostProcess的postProcessAfterInitialization方法
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    //返回实例化后的对象
    return wrappedBean;
}
```

2）查看invokeAwareMethods方法

```
private void invokeAwareMethods(final String beanName, final Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware) bean).setBeanName(beanName);
        }
        if (bean instanceof BeanClassLoaderAware) {
            ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
        }
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
    }
}
```