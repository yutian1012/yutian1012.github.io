---
title: spring框架中的英文
tags: [spring]
---

resolver：解析器/处理器

aggressively：侵略地，在java中可理解为排他独占（synchronized关键字修饰）

full-fledged：羽翼丰满的，会飞的；这里值的是比较完备的一个实现类

IOC：Inversion of Control，即“控制反转”（指的是一种设计原则）

DI：Dependency Injection，依赖注入（对控制反转的一个实现）

introspects：内省，内观；在java中可理解为查看对应的类结构定义

resort：求助，凭借； 采用的办法，求助的对象; 

dedicated to：致力于，奉献

in addition to：除...之外

Populate：居住于; 生活于; 移民于，java中表示位于该类中的代码信息

aware：意识到；觉察到；

注：在spring中大量存在的以aware为后缀的接口，实现该接口的类一般是为了让该类意识到spring相应的组件。如实现了ApplicationContextAware接口，该类就会实现setApplicationContext方法，容器实例化对象时，就会白对应的ApplicationContext注册到该对象上，那么该对象从而就能觉察到ApplicationContext对象的存在，进而利用该对象；

qualifier：合格者，已取得资格的人; 

canonical：权威的;正经篇目的;

dereference：间接引用

unified：统一的; 一元化的; 统一标准的;

traverse：横越，穿越，横贯

Eligible：合适的; 有资格当选的; 

transitive：过渡的; 转变的;