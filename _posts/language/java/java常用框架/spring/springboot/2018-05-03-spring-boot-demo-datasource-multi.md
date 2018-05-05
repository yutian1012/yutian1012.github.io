---
title: spring boot配置使用动态数据源
tags: [spring]
---

参考：https://www.cnblogs.com/fengmao/p/7538854.html

随着业务量发展，我们通常会进行数据库拆分或是引入其他数据库，从而我们需要配置多个数据源。

这里使用jdbcTemplate方式来引用访问数据源，jpa方式与之类似。

1）引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2）编辑配置文件

主数据源配置为spring.datasource.primary开头的配置，第二数据源配置为spring.datasource.secondary开头的配置。

```
# 这里使用的是jdbc-url，而不是url，一定要注意
spring.datasource.primary.jdbc-url=jdbc:mysql://localhost:3306/test
spring.datasource.primary.username=root
spring.datasource.primary.password=root
spring.datasource.primary.driver-class-name=com.mysql.jdbc.Driver

spring.datasource.secondary.jdbc-url=jdbc:mysql://localhost:3306/test2
spring.datasource.secondary.username=root
spring.datasource.secondary.password=root
spring.datasource.secondary.driver-class-name=com.mysql.jdbc.Driver
```

注：spring不能主动识别数据源，因为属性名发生了改变。spring能识别的属性名为spring.datasource.url。

如果不做相应的数据源识别类，会抛出异常信息：

```
Failed to auto-configure a DataSource: 'spring.datasource.url' is not specified and no embedded datasource could be auto-configured.
```

3）配置类

创建一个Spring配置类，定义两个DataSource用来读取application.properties中的不同配置。

@Qualifier注解是使用限定符，用来解决装配歧义问题，在相应的setbean方法上同样使用定义时的@Qualifier限定名注入的对象。

@ConfigurationProperties注解能够从配置文件application.properties中获取属性值信息，从而应用到相应的bean上。

```
package com.example.demo.config;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
public class DataSourceConfig {
    
    //数据源配置
    @Bean(name = "primaryDataSource")
    @Qualifier("primaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondaryDataSource")
    @Qualifier("secondaryDataSource")
    @Primary
    @ConfigurationProperties(prefix="spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    //jdbc匹配相应的数据源
    @Bean(name = "primaryJdbcTemplate")
    public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean(name = "secondaryJdbcTemplate")
    public JdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

4）创建数据表

在两个数据源中创建操作的数据表结构

```
-- create table `account`
DROP TABLE `account` IF EXISTS;
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `money` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
INSERT INTO `account` VALUES ('1', 'aaa', '1000');
INSERT INTO `account` VALUES ('2', 'bbb', '1000');
INSERT INTO `account` VALUES ('3', 'ccc', '1000');
```

