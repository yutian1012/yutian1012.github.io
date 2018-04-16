---
title: spring作用域
tags: [spring]
---

1）默认作用域单例

在默认情况下，Spring应用上下文中所有bean都是作为以单例（singleton）的形式创建的。也就是说，不管给定的一个bean被注入到其他bean多少次，每次所注入的都是同一个实例。

在大多数情况下，单例bean是很理想的方案。初始化和垃圾回收对象实例所带来的成本只留给一些小规模任务。

注：单例在多线程中保持无状态是将类设计成单例的标准。

注2：对于易变类型不适合创建单例。

2）spring支持的作用域

Spring定义了多种作用域，可以基于这些作用域创建bean，

a.单例（Singleton）：

在整个应用中，只创建bean的一个实例。

```
ConfigurableBeanFactory.SCOPE_SINGLETON
```

b.原型（Prototype）：

每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。

```
ConfigurableBeanFactory.SCOPE_PROTOTYPE
```

c.会话（Session）：

在Web应用中，为每个会话创建一个bean实例。

```
WebApplicationContext.SCOPe_SESSION
```

d.请求（Rquest）：

在Web应用中，为每个请求创建一个bean实例。

```
WebApplicationContext.SCOPE_REQUEST
```

3）代理模式

作用域代理能够延迟注入请求和会话作用域的bean

![](/images/spring/core/scope-proxymodel.png)