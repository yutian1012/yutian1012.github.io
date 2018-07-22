---
title: 使用dubbo搭建微服务架构-配置网关
tags: [architecture]
---

模块之间互相调用时，为了降低由网络波动带来的不确定性因素并提升系统安全性，生产环境中所有模块一般都运行在内网环境中，并单独提供一个工程作为网关服务。通过开发固定端口带来所有模块的服务，并通过拦截器验证所有外部请求以达到权限管理的目的。

外部应用有可能是APP，网站或桌面客户端，为了达到通用性，网关一般为web服务，通过Http协议提供Restful风格的API接口。

注：服务网关的作用是接收客户端的请求，隐藏内部实现，通过restful风格的API提供粗粒度的服务接口。

1）新建spring boot应用（maven module）

```
<artifactId>dubbo-gateway-demo</artifactId>

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
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

2）创建Restful接口

```
package org.sjq.dubbo.gateway;

import org.sjq.dubbo.api.IHello;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.alibaba.dubbo.config.annotation.Reference;

@RestController
public class RpcController {

    @Reference
    private IHello hello;
    
    @RequestMapping(value="/")
    public String say(){
        return hello.say("rpc");
    }
}
```

注：网关服务也是作为dubbo服务调用方来调用服务的，因此，使用@Reference注解引用相关的rpc服务。

@RestController继承自@Controller，它将自动把Resource的返回结果进行json序列化，相当与在相应的方法上添加@ResponseBody注解。并且可以为请求连接增加.json或.xml后缀来指定序列化方式。

3）创建服务启动类

```
package org.sjq.dubbo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DubboGateWayApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboGateWayApplication.class, args);
    }
}
```

4）创建并修改配置文件application.yml

```
spring:
 dubbo:
  application:
   name: gateway
  registry:
   address: zookeeper://127.0.0.1:2181
   register: false
  scan: org.sjq.dubbo.gateway

server.port: 8081
```

5）启动测试

首先前端zookeeper（/bin/zkServer.cmd），启动微服务提供方法实现注册（运行DubboServiceApplication类），最后启动网关服务（DubboGateWayApplication类）。

```
http://localhost:8081/
```