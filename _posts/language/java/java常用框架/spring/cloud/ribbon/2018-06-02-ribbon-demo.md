---
title: ribbon负载均衡实例
tags: [spring]
---

ribbon负载均衡是要配置在消费端的

1）步骤

第一步：修改microservicecloud-consumer-dept-80服务消费模块工程的pom，引入依赖

第二步：修改服务消费模块的配置文件applicatin.yml，追加eureka的服务注册地址

第三步：对ConfigBean中RestTemplate定义添加新的注解@LoadBalanced，即获得RestTemplate时加入Ribbon的配置

第四步：主启动类DeptConsumer80_App添加注解@EnableEurekaClient

第五步：修改DeptController_Consumer客户端访问类，使其通过微服务的服务名获取服务

第六步：先启动eureka的3个集群，再启动microservicecloud-provider-dept-8001并注册进eureka

第七步：启动microservicecloud-consumer-dept-80，进行测试。

2）添加依赖

修改microservicecloud-consumer-dept-80模块下的pom.xml文件

```
<!-- ribbon负载均衡 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>   
```

3）修改配置文件application.yml

这里虽然使用到了eureka，不过是从eureka中获取服务列表，而不是注册服务

```
server:
  port: 80

eureka:
 client: 
  register-with-eureka: false # 不作为服务进行注册，而是消费服务方
  service-url:
   defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

4）配置负载均衡

只需要在RestTemplate上添加@LoadBalanced注解即可（这里应该是使用代理实现的）

```
package org.sjq.test.cfgbeans;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ConfigBean {
    
    @Bean
    @LoadBalanced //spring cloud ribbon提供负载均衡
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

5）启动类上配置从eureka服务注册中心获取服务

添加@EnableEurekaClient注解

```
@SpringBootApplication
@EnableEurekaClient
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```

6）修改Controller类，通过微服务名称获取微服务地址，并访问

```
@RestController
public class DeptController_Consumer {

    //private static final String REST_URL_PREFIX="http://localhost:8001/";
    //这里不再直接使用服务地址来访问，而改成微服务名称来访问
    private static final String 
        REST_URL_PREFIX="http://MICROSERVICECLOUD-DEPT";
    
    @Autowired
    private RestTemplate restTemplate;
    
    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add", dept, Boolean.class);
    }
    ...
}
```

注：服务名称可以通过eureka注册中心获取，同时也是有服务提供者的应用名称

```
spring:
 application:
  name: microservicecloud-dept  
```

7）启动并测试

```
# 进入上传目录
cd /root/tmp
# 启动eureka服务注册中心
nohup java -jar microservicecloudEureka7001.jar >/dev/null &
nohup java -jar microservicecloudEureka7002.jar >/dev/null &
nohup java -jar microservicecloudEureka7003.jar >/dev/null &
# 启动服务提供者
nohup java -jar microservicecloudProviderDept8001.jar >/dev/null &
# 启动服务消费者
nohup java -jar microservicecloudConsumerDept80.jar >/dev/null &
```

测试消费端

```
http://47.104.19.xx/consumer/dept/get/1
```