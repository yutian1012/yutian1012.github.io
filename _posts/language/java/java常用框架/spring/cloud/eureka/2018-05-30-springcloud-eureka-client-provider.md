---
title: eureka入门实践-服务提供者注册到服务中心
tags: [spring]
---

maven模块microservicecloud-provider-dept-8001是我们定义的服务提供者，现在将服务提供者注册到eureka服务注册中心，供消费端发现并调用。

1）修改pom，引入依赖

```
<!-- 将微服务provider注册到eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

注：该依赖为eureka的客户端，启动时会将服务注册到eureka服务中心

2）修改application.yml文件

```
eureka:
 client: # 客户端注册eureka服务列表内
  service-url:
   defaultZone: http://localhost:7001/eureka
```

注：添加eureka服务注册中心地址，否则客户端不知该向哪个服务器注册

3）修改启动类

添加注解@EnableEurekaClient，服务启动时使eureka客户端生效

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient //服务启动后自动注册到eureka服务中心
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

4）测试

先启动eureka-server服务，然后启动服务提供者。

```
http://localhost:7001/
```

![](/images/spring/springcloud/eureka/eureka-server-start-provider.png)