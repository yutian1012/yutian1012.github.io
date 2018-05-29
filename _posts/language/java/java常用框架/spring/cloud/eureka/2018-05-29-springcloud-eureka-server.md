---
title: eureka入门实践-服务注册中心
tags: [spring]
---

创建maven-module（microservicecloud-eureka-7001），继承自springclouddemo父工程。

1）创建maven module模块

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.sjq.test</groupId>
        <artifactId>springclouddemo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
      </parent>
    <artifactId>microservicecloud-eureka-7001</artifactId>
</project>
```

2）添加依赖

```
<dependencies>
    <!-- eureka server服务依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
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
 port: 7001
 
eureka:
 instance:
  hostname: localhost # eureka服务端的实例名称
 client:
  register-with-eureka: false # false表示不向注册中心注册自己
  fetch-registry: false 
  service-url:
   defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka 
```

register-with-eureka参数false表示不向注册中心注册自己

fetch-registry参数false表示自己就是注册中心，职责就是维护服务实例，并不需要去检索服务

defaultZone参数表示，设置与Eureka server交互的地址查询服务和注册服务中心，其中类似el表达式的占位符就是获取该文件中定义的参数值

4）创建启动类

需要在启动类上添加@EnableEurekaServer注解，表示启动Eureka-server功能

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer//Eureka-server服务器端启动类，接收其他微服务注册进来
public class EurekaServer7001_App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class, args);
    }
}
```

5）测试

访问下面地址，如果能正常访问获取eureka页面，表示eureka服务启动成功。

```
http://localhost:7001/
```

![](/images/spring/springcloud/eureka/eureka-server-start.png)