---
title: spring cloud hystrix实例--服务熔断
tags: [spring]
---

Hystrix服务熔断（处理服务错误或异常的机制）

服务熔断，一般是某个服务故障或异常引起，类似现实世界中的保险丝，当某个异常条件被触发，直接熔断整个服务，而不是一直等到此服务超时。

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回“错误”的响应信息。当检测到该节点微服务调用响应正常后恢复调用链路。

在springcloud框架里熔断机制通过Hystrix实现，Hystrix会监控微服务间调用的状况，当失败的调用到一定的阈值，缺省是5秒内20次调用失败就会启动熔断机制。

熔断机制的注解是@HystrixCommand。

1）实例

通过在服务提供端监控异常，返回给服务消费端一个错误的响应结构，消费端看到错误的响应就会进而处理错误逻辑。

以查询Dept对象为例，正常情况下查询指定的Dept对象（页面列表提供该对象列表），但现在如果查询一个数据库中不存在的数据，默认会返回null。我们利用熔断器返回一个错误对象。

2）新建模块（类比microservicecloud-provider-dept-8001模块）

创建microservicecloud-provider-dept-hytrix-8001模块

```
<parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springclouddemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>microservicecloud-provider-dept-hytrix-8001</artifactId>
```

3）添加依赖

作为服务提供模块，依赖于microservicecloud-provider-dept-8001模块相同，并且提供Hytrix模块相关的依赖。

```
<dependencies>
    <!-- 引入api通用包，即引入了Dept部门Entity类 -->
    <dependency>
        <groupId>org.sjq.test</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <!-- 将微服务provider注册到eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <!-- 监控信息 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
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

修改打包的文件名

```
<build>
    <finalName>microservicecloudProviderDeptHytrix8001</finalName>
</build>
```

4）拷贝代码与配置文件

a.修改配置文件application.yml

只需要修改文件的instance-id，其他相关信息保持不变（如端口号，数据库配置等都不变）

```
eureka
 instance: # 自定义注册到服务中心的实例名称
  instance-id: microservicecloud-dept8001-hystrix
```

b.修改controller类

修改DeptController类，添加熔断机制，这里只对get方法进行一个熔断处理。

```
package org.sjq.test.controller;

import java.util.List;

import org.sjq.test.entities.Dept;
import org.sjq.test.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;

@RestController
public class DeptController {

    @Autowired
    private DeptService deptService;
    
    @RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
    //一旦调用服务方法失败并抛出错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod方法
    @HystrixCommand(fallbackMethod="processHystrix_Get")
    public Dept get(@PathVariable("id") Long id) {
        Dept dept =deptService.get(id);
        if(null==dept) {//为空时认为的抛出异常
            throw new RuntimeException("该ID："+id+"没有对应的信息");
        }
        return dept;
    }
    
    //熔断器处理方法-快速返回错误响应
    public Dept processHystrix_Get(@PathVariable("id") Long id) {
        return new Dept().setDeptNo(id).setDName("该ID："+id+"没有对应的信息-HystrixCommand处理")
                .setDb_source("no this database in mysql ");
    }
    ...
}
```

注：@HystrixCommand注解的使用有效类似springaop工作模式

c.修改主启动类文件

修改启动类名（方便查看区分），并在启动类添加@EnableCircuitBreaker注解

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient //服务启动后自动注册到eureka服务中心
@EnableDiscoveryClient //启动服务发现功能
@EnableCircuitBreaker//启动hytrix熔断机制的支持
public class DeptProvider8001_Hytrix_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_Hytrix_App.class, args);
    }
}
```

注：其他service和dao层类都不做修改

5）测试

```
nohup java -jar microservicecloudEureka7001.jar >/dev/null &
nohup java -jar microservicecloudEureka7002.jar >/dev/null &
nohup java -jar microservicecloudEureka7003.jar >/dev/null &
# 启动服务提供者
nohup java -jar microservicecloudProviderDeptHytrix8001.jar >/dev/null &
# 启动服务消费者
nohup java -jar microservicecloudConsumerDeptFeign80.jar >/dev/null &
```

访问服务（查看熔断器的工作）

```
# 访问1用户能够正常返回dept对象
http://47.104.249.xx/consumer/dept/get/1

# 访问100用户，由于系统中不存在，因此返回了错误的响应信息
http://47.104.249.xx/consumer/dept/get/100
```

6）存在的问题

主业务方法与熔断处理位于同一个类中，代码高度耦合。另外，要避免一个类中方法膨胀，不要有太多的方法（目前，一个业务处理方法要跟一个熔断处理方法）。

为了解耦代码，可创建一个总的负责熔断处理类，处理过程类似aop方式实现。