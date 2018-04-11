---
title: spring属性占位符
tags: [spring]
---

Spring一直支持将属性定义到外部的属性的文件中，并使用占位符值将其插入到Springbean中。在Spring装配中，占位符的形式为使用${...}包装的属性名称。

为了使用占位符，必须要配置一个PropertyPlaceholderConfigurerbean或PropertySourcesPlaceholderConfigurerbean。该类可以解析占位符，并替换成相应的属性。

解析外部属性能够将值的处理推迟到运行时，但是它的关注点在于根据名称解析来自于SpringEnvironment和属性源的属性。

1）xml中使用属性占位符

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

  <context:property-placeholder
    location="com/soundsystem/app.properties" />

  <bean class="com.soundsystem.BlankDisc"
    c:_0 = "${disc.title}"
    c:_1 = "${disc.artist}"/>

</beans>
```

注：按照这种方式，XML配置没有使用任何硬编码的值，它的值是从配置文件以外的一个源中解析得到的。

注2：Springcontext命名空间中的context:propertyplaceholder元素将会为你生成PropertySourcesPlaceholderConfigurer的bean对象。

2）自动扫描类对象动态传参

可以使用@Value注解实现参数的传入。

```
public BlankDisc(@Value("${disc.title}")String title,@Value("${disc.artist}")String artist){
    this.title=title;
    this.artist=artist;
}
```

3）如果使用java配置类动态传参还需要一个额外的配置

方法在Java中配置了PropertySourcesPlaceholderConfigurer

```
@Bean
public static PropertySourcesPlaceholderConfigurer placeholderConfigurer(){
    return new PropertySourcesPlaceholderConfigurer();
}
```

4）推荐使用PropertySourcesPlaceholderConfigurer

从Spring3.1开始，推荐使用PropertySourcesPlaceholderConfigurer，因为它能够基于SpringEnvironment及其属性源来解析占位符。