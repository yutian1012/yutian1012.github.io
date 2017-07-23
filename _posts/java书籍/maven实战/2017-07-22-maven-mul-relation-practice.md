---
title: maven多模块关联--最佳实践
tags: [maven]
---

多模块Maven项目中的聚合与继承其实是两个概念，其目的完全是不同的。前者主要是为了方便快速构建项目，后者主要是为了消除重复配置。

1）聚合关系与继承关系的比较

![](/images/book/maven/ch8/aggregator_extends.png)

在现有的实际项目中，往往会发现一个POM即是聚合POM，又是父POM，这么做主要是为了方便。

2）创建maven项目（即是聚合又是父POM）

```
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
    http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Parent</name>
    <modules>
        <module>account-email</module>
        <module>account-persist</module>
    </modules>
    ...
</project>
```

3）在父POM中定义依赖

使用dependencyManagement元素定义项目的所有模块的依赖信息

4）在父POM中定义插件

使用pluginManagement元素定义项目的所有插件信息。

5）添加新的模块

在父POM中添加新的模型，eclipse中右击项目--new--maven module。