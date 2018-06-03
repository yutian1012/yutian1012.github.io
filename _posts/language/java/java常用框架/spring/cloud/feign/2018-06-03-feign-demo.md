---
title: spring cloud feign负载均衡实例
tags: [spring]
---

实现Feign通过接口的方法调用REST服务（之前是Ribbon+RestTemplate方式实现的）。微服务名称被隐藏在了接口后方，编写代码式只需要调用接口，不用在意接口后的微服务名称。

*注：Feign进行服务调用的时候融合了Ribbon技术，所以也支持负载均衡作用。*

1）创建新模块

类比microservicecloud-consumer-dept-80模块，创建新模块microservicecloud-consumer-dept-feign-80，这样同样适用80端口。因此，两个模块实现的功能是相同的，可以相互替换。

```
<parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springclouddemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>microservicecloud-consumer-dept-feign-80</artifactId>
```

2）添加依赖

microservicecloud-consumer-dept-80模块依赖的基础上添加feign依赖.

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
    <!-- feign依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
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

修改打包部署名称

```
<build>
    <finalName>microservicecloudConsumerDeptFeign80</finalName>
</build>
```

3）拷贝代码和配置文件

拷贝启动类，并修改名称，并在启动类上添加注解

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class DeptConsumer80_Feign_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_Feign_App.class, args);
    }
}
```

注：配置文件不需要任何修改

```
server:
  port: 80

eureka:
 client: 
  register-with-eureka: false # 不作为服务进行注册，而是消费服务方
  service-url:
   defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

4）利用feign实现微服务即可（将这部分接口放到公共API模块，实现公用）

修改microservicecloud-api模块（即公共API模块），这样只要引用该模块就能具有微服务接口访问的能力（公有功能的模块化）。

a.引入Feign的依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

b.利用Feign新建微服务接口

创建DeptClientService微服务接口，该接口上使用feign提供的注解@FeignClient

```
package org.sjq.test.service;

import java.util.List;

import org.sjq.test.entities.Dept;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

//填写微服务名称
//提供接口，供访问微服务的程序使用
@FeignClient(value="MICROSERVICECLOUD-DEPT")
public interface DeptClientService {
    @RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
    public Dept get(@PathVariable("id")Long id);
    
    @RequestMapping(value="/dept/list",method=RequestMethod.GET)
    public List<Dept> list();
    
    @RequestMapping(value="/dept/add",method=RequestMethod.POST)
    public Dept add(Dept dept);
}
```

运行打包命令：

```
mvn clean install
```

5）修改controller，引入微服务接口

该类不再使用RestTemplate访问微服务，而是使用公共模块自定义的微服务接口来进行访问。

```
package org.sjq.test.controller;

import java.util.List;

import javax.annotation.Resource;

import org.sjq.test.entities.Dept;
import org.sjq.test.service.DeptClientService;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DeptController_Consumer {

    /*private static final String REST_URL_PREFIX="http://MICROSERVICECLOUD-DEPT";*/
    
    /*@Autowired
    private RestTemplate restTemplate;*/
    
    @Resource
    private DeptClientService deptClientService;
    
    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        //return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add", dept, Boolean.class);
        return deptClientService.add(dept);
    }
    
    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id")Long id) {
        //return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id, Dept.class);
        return deptClientService.get(id);
    }
    
    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        //return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list", List.class);
        return deptClientService.list();
    }
}
```

打包并部署到tomcat下

```
mvn clean package
```

6）测试微服务调用

```
nohup java -jar microservicecloudEureka7001.jar >/dev/null &
nohup java -jar microservicecloudEureka7002.jar >/dev/null &
nohup java -jar microservicecloudEureka7003.jar >/dev/null &
# 启动服务提供者
nohup java -jar microservicecloudProviderDept8001.jar >/dev/null &
nohup java -jar microservicecloudProviderDept8002.jar >/dev/null &
nohup java -jar microservicecloudProviderDept8003.jar >/dev/null &
# 启动服务消费者
nohup java -jar microservicecloudConsumerDeptFeign80.jar >/dev/null &
```

浏览器访问服务消费者，查看是否轮询访问各个服务