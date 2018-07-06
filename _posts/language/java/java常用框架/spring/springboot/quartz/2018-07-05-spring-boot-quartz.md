---
title: spring boot 整合quartz
tags: [spring]
---

参考：https://blog.csdn.net/u012907049/article/details/73801122/

1）spring boot项目中引入jar包

```
<!-- 定时器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
 </dependency>
```

查看依赖的版本号

2）根据依赖的版本号下载相应的数据库文件

官网地址：http://www.quartz-scheduler.org/downloads/

下载压缩文件后，在docs/dbTables目录下有建表需要的sql语句

```
drop database if exists spider_quartz;

create database spider_quartz;
```

运行相关的数据库建表语言即可完成数据库的安装。

3）创建配置文件

在resources目录下创建quartz.properties文件

```

```