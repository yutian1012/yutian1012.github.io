---
title: spring的环境迁移
tags: [spring]
---

在开发软件的时候，有一个很大的挑战就是将应用程序从一个环境迁移到另外一个环境。开发阶段中，某些环境相关做法可能并不适合迁移到生产环境中，甚至即便迁移过去也无法正常工作。数据库配置、加密算法以及与外部系统的集成是跨环境部署时会发生变化的几个典型例子。

1）spring profile方式

spring profile能够根据环境决定该创建哪个bean和不创建哪个bean（创建后放置到容器中，但在编译期都会编译）。不过Spring并不是在构建的时候做出这样的决策，而是等到运行时再来确定。这样的结果就是同一个部署单元（可能会是WAR文件）能够适用于所有的环境，没有必要进行重新构建。

2）maven选择编译方式

针对各种环境都单独有各自的配置类（或XML文件），然后在构建阶段（可能会使用Maven的profiles）确定要将哪一个配置编译到可部署的应用中。这种方式的问题在于要为每种环境重新构建应用。

3）spring profile要求

要使用profile，你首先要将所有不同的bean定义整理到一个或多个profile之中，在将应用部署到每个环境时，要确保对应的profile处于激活（active）的状态。没有指定profile的bean始终都会被创建，与激活哪个profile没有关系。

a.java配置中使用@Profile注解指定某个bean属于哪一个profile。

b.激活profile

4）@Profile注解

在Spring3.1中，只能在类级别上使用@Profile注解。不过，从Spring3.2开始，你也可以在方法级别上使用@Profile注解，与@Bean注解一同使用。这样的话，就能将这两个bean的声明放到同一个配置类之中，

从Spring4开始，@Profile注解进行了重构，使其基于@Conditional和Condition实现。