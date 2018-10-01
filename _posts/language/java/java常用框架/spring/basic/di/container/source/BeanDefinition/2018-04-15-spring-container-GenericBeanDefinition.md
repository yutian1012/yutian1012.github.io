---
title: spring中GenericBeanDefinition对象的理解
tags: [spring]
---

GenericBeanDefinition是一站式的标准beanDefinition，除了具有指定类、可选的构造参数值和属性参数这些其它beanDefinition一样的特性外，它还具有通过parenetName属性来灵活设置parent beaDefinition。

1）定义信息

```
public class GenericBeanDefinition extends AbstractBeanDefinition {
    private String parentName;
    ...
}
```

通常，GenericBeanDefinition用来注册用户可见的beanDefinition。可见的beanDefinition意味着可以在该类beanDefinition上定义post-processor来对bean进行操作，甚至为配置parentname做扩展准备。

注：RootBeanDefinition/ChildBeanDefinition用来预定义具有parent/child关系的beanDefinition。