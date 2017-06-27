---
title: spring扫描通配符
tags: [spring]
---

参考：https://zhidao.baidu.com/question/458949129873516885.html

参考：http://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/beans.html#beans-annotation-config

### base-package扫描语法

```
<context:component-scan base-package="com.xxx"/>
```

实现类：ComponentScanBeanDefinitionParser

参考：https://my.oschina.net/HeliosFly/blog/203149