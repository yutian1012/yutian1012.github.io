---
title: spring cloud 服务消费者
tags: [spring]
---

创建maven-module（microservicecloud-consumer-dept-80），继承自springclouddemo父工程。

1）创建maven module模块

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springclouddemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>microservicecloud-consumer-dept-80</artifactId>
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
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
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
</dependencies>
```

3）创建配置文件application.yml

```
server:
  port: 80
```

4）创建配置类

springboot使用配置类的方式代替传统的applicationContext.xml方式的配置

```
package org.sjq.test.cfgbeans;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ConfigBean {
    
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

5）补充RestTemplate类

RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类。是spring提供的用于访问Rest服务的客户端模板工具集（类比Jdbc操作有JdbcTemplate模板类帮助）。

使用RestTemplate访问restfule接口非常简单，只需要提供三个参数即可实现rest接口的访问。参数（url、requestMap，ResponseBean.class）分别代表Rest请求地址，请求参数和http响应转换成的对象类型。

6）创建controller层接口

```
package org.sjq.test.controller;

import java.util.List;

import org.sjq.test.entities.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class DeptController_Consumer {

    private static final String REST_URL_PREFIX="http://localhost:8001/";
    
    @Autowired
    private RestTemplate restTemplate;
    
    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        //发送post请求
        return restTemplate.postForObject(REST_URL_PREFIX+"dept/add", dept, Boolean.class);
    }
    
    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id")Long id) {
        //发送get请求
        return restTemplate.getForObject(REST_URL_PREFIX+"dept/get/"+id, Dept.class);
    }
    
    @SuppressWarnings("unchecked")
    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        return restTemplate.getForObject(REST_URL_PREFIX+"dept/list", List.class);
    }
}
```

7）创建微服务启动类

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```

8）测试

```
# 列表查看
http://localhost/consumer/dept/list
# 添加
http://localhost/consumer/dept/add?dName=2018BigData
# 获取
http://localhost/consumer/dept/get/6
```