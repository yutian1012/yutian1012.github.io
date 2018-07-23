---
title: 使用dubbo搭建微服务架构-服务降级@Reference注解
tags: [architecture]
---

在reference注解，有一个参数mock，该参数有四个值，false,default,true,或者Mock类的类名。分别代表如下含义:

```
false，不调用mock服务。
true，当服务调用失败时，使用mock服务。
default，当服务调用失败时，使用mock服务。
force，强制使用Mock服务(不管服务能否调用成功)。
```
