---
title: spring事务配置方式
tags: [spring]
---

参考：http://www.cnblogs.com/ms-grf/p/7449188.html

### aop声明式事务配置

### 注解式事务配置

### 两种事务方式共存

参考：http://blog.csdn.net/jeep_ouc/article/details/43056977

配置事务的先后顺序

```
<tx:annotation-driven transaction-manager="txManager" order="0"/>
<aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"  order="1"/>
```

问题：注解事务是基于aop方式实现的吗？