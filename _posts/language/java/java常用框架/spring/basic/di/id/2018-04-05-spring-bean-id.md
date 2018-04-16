---
title: spring的ID字符串
tags: [spring]
---

Spring应用上下文中所有的bean都会给定一个ID。默认为将类名的第一个字母变为小写。

1）注解方式指定ID

a.将ID值作为参数传入@Component注解

b.使用Java依赖注入规范提供的@Name注解也可以实现。

2）xml方式指定ID

```
<bean id="xxx" class="soundsystem.SgtPeppers"/>
```

注：如果没有明确给定ID，这个bean将会根据全限定类名来进行命名。因此这里bean的ID将会是"soundsystem.SgtPeppers#0"。

3）Java配置方式