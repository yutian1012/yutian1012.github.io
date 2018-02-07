---
title: activiti用户指南
tags: [activiti]
---

参考：https://www.activiti.org/userguide/

### 环境准备

1）源码下载地址：

https://github.com/Activiti/Activiti

2）eclipse的选择

If you would like to use the Activiti Designer then you need Eclipse Kepler or Luna.

安装插件，Help--Install New Software

```
Name:Activiti BPMN 2.0 designer
Location:http://activiti.org/designer/update/
```

3）数据库连接

mybatis处理大量操作时效果不好，建议使用jdbc方式或datasource方式来操作数据库。

注：Our benchmarks have shown that the MyBatis connection pool is not the most efficient or resilient when dealing with a lot of concurrent requests.

databaseSchemaUpdate类型的设置：false (default)，true，create-drop。

4）数据库脚本

activiti-engine\src\main\resources\org\activiti\db\目录下存在创建数据库的脚本。

针对mysql数据库版本，如果版本小于等于5.6.3建议是mysql55脚本创建（主要是为了处理timestamps的日期字段）。大于5.6.4版本的可以使用mysql的脚本创建。

5）activiti数据库表结构版本更新

在activiti.cfg.xml中配置databaseSchemaUpdate属性值为true，则会根据classpath下的相关activiti的jar更新数据库表的版本，使其一致。

6）日志记录

slf4j是一个日志的桥接模块。添加log4j作为日志记录模块

```
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```