---
title: spring配置方式
tags: [spring]
---

1）使用配置类方式

```
import org.springframework.context.annotation.ComponentScan;
importorg.springframework.context.annotation.Configuration;
@Configuration
@ComponentScan//bean通过配置自动扫描定义到容器中
public class CDPlayerConfig{
    //bean定义到容器方式
    @Bean
    public Knight knight() {
        //bean的依赖注入方式
        return new BraveKnight(quest());
    }
    
    @Bean
    public Quest quest() {
        return new SlayDragonQuest(System.out);
    }
}
```

注：ComponentScan会扫描配置的包，及其该包下的子包。

2）使用xml方式

使用相应的标签需要引入相关的命名空间

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"

    xsi:schemaLocation=
    "http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context.xsd">
  
  <!-- 自动扫描方式 -->
  <context:component-scan base-package="soundsystem"/>

  <!-- 标签定义bean -->
  <bean id="knight" class="sia.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

</beans>
```