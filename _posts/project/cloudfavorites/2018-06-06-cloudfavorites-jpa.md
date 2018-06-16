---
title: github开源项目云收藏（jpa操作对象）
tags: [project]
---

1）引入Jpa相关jar文件

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2）配置数据源

编辑application.properties文件

```
######  data config ###### 
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql=true

######  db config ######
spring.datasource.url=jdbc:mysql://127.0.0.1/favorites_demo?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

创建mysql数据库

```
drop database if exists favorites_demo;
create database favorites_demo;
```

3）引入lombak，可以简化javaBean对象属性的定义内容

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```