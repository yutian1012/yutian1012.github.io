---
title: spring的bean标签属性lookup-method
tags: [spring]
---

参考：https://www.cnblogs.com/ViviChan/p/4981619.html

Spring会对bean指定的class做动态代理，代理lookup-method标签中name属性所指定的方法，返回bean属性指定的bean实例对象（内部调用了CGLIB生成的动态代理类的方法）。