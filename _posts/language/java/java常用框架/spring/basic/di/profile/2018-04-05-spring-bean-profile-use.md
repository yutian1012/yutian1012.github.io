---
title: spring的环境迁移实例
tags: [spring]
---

要使用profile，你首先要将所有不同的bean定义整理到一个或多个profile之中，在将应用部署到每个环境时，要确保对应的profile处于激活（active）的状态。没有指定profile的bean始终都会被创建，与激活哪个profile没有关系。

1）java配置中使用@Profile注解指定某个bean属于哪一个profile。

a.java方式配置profile

使用@Profile注解

```
//测试环境下的配置信息
package com.myapp;
import javax.activation.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;

@Configuration
@Profile("dev")
public class DevelopmentProfileConfig{
    @Bean(destroyMethod="shutdown")
    public DataSource dataSource(){
        return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2)
        .addScript("classpath:schema.sql")
        .addScript("classpath:test-data.sql").build();
    }
}

//生产环境下的配置信息
package com.myapp;
import javax.activation.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jndi.JndiObjectFactoryBean;

@Configuration
@Profile("prod")
public class ProductionProfileConfig{
    @Bean
    public DataSource dataSource(){
        JndiObjectFactoryBean jndiObjectFactoryBean=new JndiObjectFactoryBean();
        jndiObjectFactoryBean.setJndiName("jdbc/myDS");
        jndiObjectFactoryBean.setResourceRef(true);
        jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
        return(DataSource)jndiObjectFactoryBean.getObject();
    }
}
```

b.xml中配置profile，

在beans标签中配置profile属性

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"

    xsi:schemaLocation=
        "http://www.springframework.org/schema/jdbc
        http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
        profile="dev">
    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:schema.sql"/>
        <jdbc:script ocation="classpath:test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

2）激活profile

Spring在确定哪个profile处于激活状态时，需要依赖两个独立的属性：spring.profiles.active和spring.profiles.default。

如果设置了spring.profiles.active属性的话，那么它的值就会用来确定哪个profile是激活的。但如果没有设置spring.profiles.active属性的话，那Spring将会查找spring.profiles.default的值。如果spring.profiles.active和spring.profiles.default均没有设置的话，那就没有激活的profile，因此只会创建那些没有定义在profile中的bean。

3）激活方式

a.作为DispatcherServlet的初始化参数；

b.作为Web应用的上下文参数；

c.作为JNDI条目；

d.作为环境变量；

e.作为JVM的系统属性；

f.在集成测试类上，使用@ActiveProfiles注解设置。

4）最佳实践

