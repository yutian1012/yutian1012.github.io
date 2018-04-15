---
title: spring容器的ignoreDependencyInterface方法
tags: [spring]
---

AbstractAutowireCapableBeanFactory父类构造

```
public AbstractAutowireCapableBeanFactory() {
    super();//继续上调默认构造器
    ignoreDependencyInterface(BeanNameAware.class);
    ignoreDependencyInterface(BeanFactoryAware.class);
    ignoreDependencyInterface(BeanClassLoaderAware.class);
}
```

如何理解忽略这些回调类？