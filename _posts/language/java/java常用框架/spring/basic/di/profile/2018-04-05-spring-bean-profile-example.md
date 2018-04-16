---
title: spring设置profile
tags: [spring]
---

要使用profile，你首先要将所有不同的bean定义整理到一个或多个profile之中，在将应用部署到每个环境时，要确保对应的profile处于激活（active）的状态。没有指定profile的bean始终都会被创建，与激活哪个profile没有关系。

1）java配置类设置profile

使用@Profile注解，即可以在配置类上，也可以在配置类的方法上

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

注：另外@Profile注解也可以配置在配置类的创建bean对象方法上。

2）xml中配置profile，

在beans标签中存在profile属性，通过设置该值来控制profile类型

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

3）在类上添加注解，

自动扫描功能会根据类中定义的profile，并根据激活的profile对bean进行实例化。

```
@Component
@Profile("prod")
public class ProductService{}
```