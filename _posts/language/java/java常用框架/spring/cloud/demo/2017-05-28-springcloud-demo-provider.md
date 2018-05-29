---
title: spring cloud 服务提供者
tags: [spring]
---

创建maven-module（microservicecloud-provider-dept-8001），继承自springclouddemo父工程。

1）创建maven module模块

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.sjq.test</groupId>
        <artifactId>springclouddemo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
      </parent>
    <artifactId>microservicecloud-provider-dept-8001</artifactId>
</project>
```

2）添加依赖

```
<dependencies>
    <!-- 引入api通用包，即引入了Dept部门Entity类 -->
    <dependency>
        <groupId>org.sjq.test</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>    
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!-- 修改后立即生效，热部署 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>springloaded</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

3）创建配置文件application.yml

该配置文件配置了，内嵌servlet容器的服务端口，mybatis的配置信息，微服务名称以及数据源。

```
server:
 port: 8001

mybatis:
 config-location: classpath:mybatis/mybatis.cfg.xml # mybatis配置文件所在路径
 type-aliases-package: org.sjq.test.entities            # 所有Entity别名类所在包
 mapper-locations: classpath:mybatis/mapper/**/*.xml    # mapper映射文件

spring:
 application:
  name: microservicecloud-dept                      # 对微服务而已非常重要
 datasource:
  type: com.alibaba.druid.pool.DruidDataSource      # 当前数据源操作类型
  driver-class-name: org.gjt.mm.mysql.Driver        # mysql驱动包
  url: jdbc:mysql://localhost:3306/cloudDB01        #  数据库名称
  username: root
  password: root
  dbcp2:
   min-idle: 5                                  # 数据库连接池的最小维持连接数
   initial-size: 5                              # 初始化连接数
   max-total: 5                                 # 最大连接数 
   max-wait-millis: 200                         # 等待连接获取的最大超时时间     
```

注：值和key之间使用空格分隔

4）整合mybaits

查看文档springcloud-demo-provider-mybatis

5）创建service接口与实现类

在Service实现类中引入Dao接口，实现对数据库的访问

```
package org.sjq.test.service;

import java.util.List;

import org.sjq.test.entities.Dept;

public interface DeptService {
    public boolean add(Dept dept);
    
    public Dept get(Long id);
    
    public List<Dept> list();
}
```

实现类DeptServiceImpl

```
package org.sjq.test.service;

import java.util.List;

import org.sjq.test.dao.DeptDao;
import org.sjq.test.entities.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DeptServiceImpl implements DeptService{

    @Autowired
    private DeptDao dao;
    
    @Override
    public boolean add(Dept dept) {
        return dao.addDept(dept);
    }

    @Override
    public Dept get(Long id) {
        return dao.findById(id);
    }

    @Override
    public List<Dept> list() {
        return dao.findAll();
    }
}
```

6）创建controller层类

在controller类中注入service层接口，实现对相应业务逻辑类的调用。controller使用restful方式定义api接口，返回json数据，从而实现前后端分离。

```
package org.sjq.test.controller;

import java.util.List;

import org.sjq.test.entities.Dept;
import org.sjq.test.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DeptController {

    @Autowired
    private DeptService deptService;
    
    @RequestMapping(value="/dept/add",method=RequestMethod.POST)
    public boolean add(@RequestBody Dept dept) {
        return deptService.add(dept);
    }
    
    @RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id) {
        return deptService.get(id);
    }
    
    @RequestMapping(value="/dept/list",method=RequestMethod.GET)
    public List<Dept> list() {
        return deptService.list();
    }
}
```

7）创建启动springboot微服务类

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

8）测试访问

```
http://localhost:8001/dept/list
```