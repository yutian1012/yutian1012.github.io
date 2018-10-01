---
title: spring容器的ignoreDependencyInterface方法
tags: [spring]
---

在DefaultListableBeanFactory对象初始化时会调用父类AbstractAutowireCapableBeanFactory类构造函数。

1）AbstractAutowireCapableBeanFactory构造函数

```
public AbstractAutowireCapableBeanFactory() {
    super();//继续上调默认构造器
    ignoreDependencyInterface(BeanNameAware.class);
    ignoreDependencyInterface(BeanFactoryAware.class);
    ignoreDependencyInterface(BeanClassLoaderAware.class);
}
```

如何理解忽略这些回调类？