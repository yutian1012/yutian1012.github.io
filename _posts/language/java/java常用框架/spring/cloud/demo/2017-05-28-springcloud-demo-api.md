---
title: spring cloud 公共API模块
tags: [spring]
---

创建maven module（microservicecloud-api），继承自springclouddemo父工程。

1）创建maven module模块

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.sjq.test</groupId>
        <artifactId>springclouddemo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <!-- 当前module自己的artifactId -->
    <artifactId>microservicecloud-api</artifactId>
</project>
```

创建完成后，在父类中会出现modules标签中添加上相关的模块

```
<modules>
    <module>microservicecloud-api</module>
</modules>
```

注：这里就是maven的继承策略

2）添加依赖

```
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

3）创建公共实体类Dept

```
package org.sjq.test.entities;

import java.io.Serializable;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;

//部门实体类
@AllArgsConstructor
@NoArgsConstructor
@Data
@Accessors(chain=true)//提供链式方式
public class Dept implements Serializable{
    private Long deptNo;//主键
    private String dName;//部门名称
    //来自哪个数据库，因为微服务架构可以一个服务对应一个数据库
    //同一个信息被存储到不同的数据库中
    private String db_source;
}
```

4）安装到本地仓库

定位到microservicecloud-api目录下，运行命令

```
mvn clean install
```

注：安装完成后，只需要在相应的模块中引入该GAV即可实现jar包的依赖引入。