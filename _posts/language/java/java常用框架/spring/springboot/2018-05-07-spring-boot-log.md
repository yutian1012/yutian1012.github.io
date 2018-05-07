---
title: spring boot配置使用日志
tags: [spring]
---

参考：https://blog.csdn.net/inke88/article/details/75007649

spring-boot默认情况下，会用Logback来记录日志，并用INFO级别输出到控制台。可以在项目的依赖包中看到相应的jar文件。

1）日志依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

实际开发中不需要直接添加该依赖。当引入spring-boot-starter获取其他的构建时，其中就包含了spring-boot-starter-logging。

2）日志级别

```
TRACE < DEBUG < INFO < WARN < ERROR < FATAL
```

从低到高，输出的信息也越来越少，即TRACE输入的信息是最多的。

在application.properties文件中配置输出级别

```
debug=true
```

3）程序中输出日志

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private final Logger logger=LoggerFactory.getLogger(MigrateDao.class);

...
//输出日志debug
if(logger.isDebugEnabled()){
    logger.debug("update the data success!");
}
```

4）使用lombok可以简化日志

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

lombok可以通过注解的形式简化Logger对象的生成，从而不用在每个类中都使用LoggerFactory.getLogger获取logger对象。