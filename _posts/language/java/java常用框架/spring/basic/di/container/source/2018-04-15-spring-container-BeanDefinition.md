---
title: spring中BeanDefinition对象的理解
tags: [spring]
---

BeanDefinition相当于一个数据结构，这个数据结构的生成过程是根据解析的resource资源对象中的bean定义而来的，这些bean在Spirng IoC容器内部表示成了的BeanDefintion这样的数据结构，IoC容器对bean的管理和依赖注入的实现都是通过操作BeanDefinition来进行的。