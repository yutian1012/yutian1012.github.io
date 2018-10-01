---
title: spring容器初始化BeanFactory对象
tags: [spring]
---

obtainFreshBeanFactory方法主要用于初始化beanFactory对象（DefaultListableBeanFactory实例），设置beanFactory的setSerializationId属性，并设置属性refreshed为true。

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //设置属性等信息
    refreshBeanFactory();
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (logger.isDebugEnabled()) {
        logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
    }
    return beanFactory;
}
```

查看GenericApplicationContext类的refreshBeanFactory方法

```
@Override
protected final void refreshBeanFactory() throws IllegalStateException {
    if (this.refreshed) {
        throw new IllegalStateException(
                "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
    }
    this.beanFactory.setSerializationId(getId());
    this.refreshed = true;
}
```

注：由于AnnotationConfigApplicationContext类继承GenericApplicationContext，而在GenericApplicationContext中包含beanFactory成员属性（私有）。因此AnnotationConfigApplicationContext就拥有了beanFacotry，通过getBeanFactory方法既可以获取beanFactroy实例。