---
title: spring表达式
tags: [spring]
---

Spring3引入了Spring表达式语言（SpringExpressionLanguage，SpEL），它能够以一种强大和简洁的方式将值装配到bean属性和构造器参数中，在这个过程中所使用的表达式会在运行时计算得到值。使用SpEL，你可以实现超乎想象的装配效果，这是使用其他的装配技术难以做到的（甚至是不可能的）。

1）spEl表达式提供的特性

a.使用bean的ID来引用bean；

b.调用方法和访问对象的属性；

c.对值进行算术、关系和逻辑运算；

d.正则表达式匹配；

e.集合操作。

2）其他用途

SpEL能够用在依赖注入以外的其他地方。

a.Spring Security支持使用SpEL表达式定义安全限制规则。

b.在Spring MVC应用中使用Thymeleaf模板作为视图的话，那么这些模板可以使用SpEL表达式引用模型数据。

3）使用方式

SpEL表达式需要使用#{...}来引用。