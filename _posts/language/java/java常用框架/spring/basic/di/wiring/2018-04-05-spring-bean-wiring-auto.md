---
title: spring的自动装配
tags: [spring]
---

Spring从两个角度来实现自动化装配：

1）组件扫描（component scanning）：（将bean设置到上下文容器中）

a.通过@ComponentScan注解扫描组件

Spring会自动发现应用上下文中所创建的bean。多个扫描包之间使用逗号分隔，会对base-package包或者子包下的所有的进行java类进行扫描，并把匹配的java类注册成bean。

```
import org.springframework.context.annotation.ComponentScan;
importorg.springframework.context.annotation.Configuration;
@Configuration
@ComponentScan
public class CDPlayerConfig{}
```

b.通过xml扫描组件

```
<context:component-scan base-package="soundsystem"/>
```

注：component-scan位于context命名空间下，需要引入相应的命名空间。

c.类型安全的扫描设置

上述所设置的基础包是以String类型表示的。这种方法是类型不安全（nottype-safe）的。如果你重构代码的话，那么所指定的基础包可能就会出现错误。除了将包设置为简单的String类型之外，@ComponentScan还提供了另外一种方法，那就是将其指定为包中所包含的类或接口，

```
@ComponentScan(basePackageClasses={CDPlayer.class,DVDPlayer.class})
```

注：会将类所在的包作为基础包进行扫描

2）自动装配（autowiring）：（将符合的依赖注入到bean，完成依赖注入）

Spring自动满足bean之间的依赖。spring会在应用上下文中寻找匹配某个bean需求的其他bean。

a.使用@Autowired注解进行自动装配，该注解是spring提供的。

b.使用@Inject注解进行自动装配，该注解是有java规范提供的