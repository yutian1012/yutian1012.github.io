---
title: 项目的继承依赖
tags: [kangaroo]
---

尽量避免使用框架的依赖，将controller，service，dao的底层依赖脱离开特定的框架。如使用java提供的注解，而不是spring提供的注解，校验也使用java提供的注解，而不是hibernate-validator注解。spring包扫描也支持java的注解。