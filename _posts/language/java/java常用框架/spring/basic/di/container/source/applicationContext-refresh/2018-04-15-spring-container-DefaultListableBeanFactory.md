---
title: spring容器DefaultListableBeanFactory
tags: [spring]
---

关键类DefaultListableBeanFactory

在BeanFactory子类中有一个DefaultListableBeanFactory类，它包含了基本Spirng IoC容器所具有的重要功能，开发时不论是使用BeanFactory系列还是ApplicationContext系列来创建容器基本都会使用到DefaultListableBeanFactory类，可以这么说，在spring中实际上把它当成默认的IoC容器来使用。