---
title: spring的装配混合
tags: [spring]
---

在Spring中，这些配置方案都不是互斥的。可以将JavaConfig的组件扫描和自动装配和/或XML配置混合在一起。在自动装配时，它并不在意要装配的bean来自哪里。自动装配的时候会考虑到Spring容器中所有的bean，不管它是在JavaConfig或XML中声明的还是通过组件扫描获取到的。

1）在配置类中引入xml配置

使用@ImportResource注解来引入xml配置信息，使用@Import注解来引入其他配置类

```
package soundsystem;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.ImportResource;
@Configuration
//引入配置类
@Import(CDPlayerConfig.class)
//引入xml配置文件
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig{}
```

2）在xml配置中引入java配置类

使用import标签引入其他xml配置信息，使用

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"

    xsi:schemaLocation=
        "http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 引入xml配置文件 -->
    <import resource="cd-config.xml"/>
    <!-- 引入配置类，只需要简单的以bean的信息配置即可 -->
    <bean class="soundsystem.CDConfig" />
    <bean id="cdPlayer" class="soundsystem.CDPlayer"
        c:cd-ref="compactDisc"/>
    </beans>
```

3）最佳实践

不管使用JavaConfig还是使用XML进行装配，通常都会创建一个根配置（rootconfiguration），这个配置会将两个或更多的装配类或XML文件组合起来。在根配置中启用组件扫描（通过context:component-scan或@ComponentScan）。