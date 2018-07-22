---
title: 使用dubbo搭建微服务架构-maven搭建
tags: [architecture]
---

整个项目工程都使用spring boot进行搭建，因此需要在父项目中定义相关的依赖。

1）创建maven project作为一个父项目

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.sjq.test</groupId>
  <artifactId>dubbo-demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>

  <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring-boot.version>1.5.9.RELEASE</spring-boot.version>
        <lombok.version>1.16.18</lombok.version>
  </properties>

  <dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${spring-boot.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
  </dependencyManagement>
  
  <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </pluginManagement>
  </build>
</project>
```

注：这是一个pom类型的maven项目

2）新建接口工程dubbo-api-demo

该接口工程用于定义微服务端的公共接口，服务于服务提供方和消费方。

在上面的父项目下创建maven module模块，作为公共接口工程。

```
<parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>dubbo-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>dubbo-api-demo</artifactId>
```

创建一个接口IHello

```
package org.sjq.dubbo.api;

public interface IHello {
    public String say(String msg);
}
```

3）创建服务工程dubbo-service-demo

创建maven module模块，并引入相关的接口和dubbo依赖

```
<dependencies>
    <dependency>
        <groupId>${project.groupId}</groupId>
        <artifactId>dubbo-api-demo</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>io.dubbo.springboot</groupId>
        <artifactId>spring-boot-starter-dubbo</artifactId>
        <version>1.0.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

创建接口的服务实现类

```
package org.sjq.dubbo.service;

import org.sjq.dubbo.api.IHello;

import com.alibaba.dubbo.config.annotation.Service;

@Service
public class helloImpl implements IHello{

    @Override
    public String say(String msg) {
        System.out.println("hello:"+msg);
        return msg;
    }
}
```

创建服务启动类

```
package org.sjq.dubbo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DubboServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboServiceApplication.class, args);
    }
}
```

配置application.yml文件，在配置文件中定义应用名称，注册中心地址，日志等相关信息

```
spring:
 dubbo:
  application:
   name: provider
  registry:
   address: zookeeper://127.0.0.1:2181
  protocol:
   name: dubbo
   port: -1
  scan: org.sjq.dubbo.service
```

注：scan属性中指定了@Service注解所在的包，dubbo扫描到接口后，会暴露该服务接口。

4）服务消费方

创建maven module模块，并引入相关的接口和dubbo依赖

```
<dependencies>
    <dependency>
        <groupId>${project.groupId}</groupId>
        <artifactId>dubbo-api-demo</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>io.dubbo.springboot</groupId>
        <artifactId>spring-boot-starter-dubbo</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

注：依赖与服务提供者是相同的。

编写远程调用dubbo服务

```
package org.sjq.dubbo.client;

import org.sjq.dubbo.api.IHello;
import org.springframework.stereotype.Component;

import com.alibaba.dubbo.config.annotation.Reference;

@Component
public class InvokeService {
    @Reference
    public IHello hello;
}
```

编写spring boot服务启动类

```
package org.sjq.dubbo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DubboClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboClientApplication.class, args);
    }
}
```

新建并编辑配置文件applicatin.yml

```
spring:
 dubbo:
  application:
   name: consumer
  registry:
   address: zookeeper://127.0.0.1:2181
  scan: org.sjq.dubbo.client
```

注：与服务提供方的配置一致，指定zookeeper注册中心地址，并提供不同的application.name便可完成调用端的配置。

编写测试类

```
package org.sjq.dubbo.client;

import static org.junit.Assert.*;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class InvokeServiceTest {
    
    @Resource
    private InvokeService invokeService;

    @Test
    public void test() {
        String result=invokeService.hello.say("www.baidu.com");
        assertTrue("www.baidu.com".equals(result));
    }
}
```

5）启动测试

首先前端zookeeper（/bin/zkServer.cmd），启动微服务提供方法实现注册（运行DubboServiceApplication类），最后启动调用端的测试代码（运行InvokeServiceTest类）。

注：目前服务端和客户端调用以基本完成。