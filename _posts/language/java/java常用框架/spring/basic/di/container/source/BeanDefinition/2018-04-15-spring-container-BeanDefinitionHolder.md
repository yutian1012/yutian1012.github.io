---
title: spring中BeanDefinitionHolder对象的理解
tags: [spring]
---

参考：https://blog.csdn.net/u011179993/article/details/51598567

持有一个BeanDefinition，名称，和别名数组。在Spring内部，它用来临时保存BeanDefinition来传递BeanDefinition。

注：临时保存

1）类定义信息

```
private final BeanDefinition beanDefinition;
    private final String beanName;
    private final String[] aliases;
    ...
}
```