---
title: spring容器的HierarchicalBeanFactory接口
tags: [spring]
---

1）介绍（查看类的英文注释）

提供父容器的访问功能，至于父容器的设置，需要找ConfigurableBeanFactory的setParentBeanFactory。

2）接口方法

```
//获取父BeanFactory容器
BeanFactory getParentBeanFactory();

//判断当前容器是否包含bean，在当前容器中查找，不从父容器中查找
boolean containsLocalBean(String name);
```

注：从方法上可以看出，提供了父容器的获取，及容器是有层级概念的。