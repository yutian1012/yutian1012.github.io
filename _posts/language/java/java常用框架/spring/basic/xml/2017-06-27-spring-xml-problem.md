---
title: spring解析xml时常见错误
tags: [spring]
---

### Error occured processing XML

1）spring配置文件xml错误的地方

```
<context:component-scan base-package="org.ipph.web.exception"/>
```

spring配置文件xml提示错误

```
Error occured processing XML '[Ljava.lang.String; cannot be cast to java.lang.String'. See Error Log for more details
```

![](/images/java_structure/spring/xml/processxmlerror1.png)

2）原因

在exception包下存在@ControllerAdvice注解修饰的类。另外，该注解被@Component修饰，因此可以被component-scan扫描到。

3）修改

```
<context:component-scan base-package="org.ipph.web.exception.*"/>
```