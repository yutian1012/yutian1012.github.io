---
title: spring cloud hystrix dashboard
tags: [spring]
---

除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用健康（Hystrix-Dashboard）。Hystrix会持续地记录所有通过Hystrix发起的请求执行信息，并以统计报表和图形的形式展示给用，包括每秒执行多少请求，多少成功，多少失败等等。

注：是通过hystrix发起的请求，这个是硬性条件限制，也就是说服务器提供者方必须使用了hystrix。

Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。spring-cloud也提供了Hystrix-Dashboard的整合，对监控内容转化成可视化界面。

1）新建工程

新建maven模块：microservicecloud-consumer-hystrix-dashboard。

```
<parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springclouddemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>microservicecloud-consumer-hystrix-dashboard</artifactId>
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
    <!-- hytrix dashboard -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
    </dependency>
</dependencies>
```

打包jar命名

```
<build>
    <finalName>microservicecloudConsumerHystrixDashboard</finalName>
</build>
```

3）创建配置文件

创建配置文件application.yml，只需要配置服务的端口即可

```
server:
  port: 8901
```

4）创建主启动类

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumer_Dashboard_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_Dashboard_App.class, args);
    }
}
```

5）需要在被监控的微服务中添加依赖

这些微服务要心甘情愿的被监控，需要引入相关的监控模块。将微服务提供者（microservicecloud-provider-dept-hytrix-8001），并且服务要使用熔断机制，否则不能正确查看到服务的调用信息。因为该监控就是为了查看熔断信息而存在的。

添加actuator监控依赖。

```
<!-- 监控信息 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

6）单独运行监控查看

运行主启动类DeptConsumer_Dashboard_App

```
# 访问监控服务地址
http://localhost:8901/hystrix
```

7）在linux系统上运行

```
nohup java -jar microservicecloudEureka7001.jar >/dev/null &
nohup java -jar microservicecloudEureka7002.jar >/dev/null &
nohup java -jar microservicecloudEureka7003.jar >/dev/null &
# 启动服务监控服务
nohup java -jar microservicecloudConsumerHystrixDashboard.jar >/dev/null &
# 启动服务提供者
nohup java -jar microservicecloudProviderDeptHytrix8001.jar >/dev/null &
```

注：监控的信息必须是使用了hystrix熔断机制的服务，普通的服务是无法查看的。

8）直接在浏览器中访问监控数据

查看单个微服务的监控信息

```
# 监控8001服务调用信息，访问微服务的地址+hystrix.stream后缀
http://47.104.249.xxx:8001/hystrix.stream
```

无法看到没有使用hystrix机制的服务调用信息

```
# 启动没有使用hystrix机制的服务提供者--不能看到调用信息
nohup java -jar microservicecloudProviderDeptHytrix8002.jar >/dev/null &
```

9）图像化的方式查看

```
# 访问hystrix服务
http://47.104.249.xxx:8901/hystrix

# 在页面中相应的位置输入查询服务，统计间隔等信息
监控服务地址：http://47.104.249.xxx:8001/hystrix.stream
Delay: 2000ms
Title: demo001
```

![](/images/spring/springcloud/hystrix/hystrix-server.png)

![](/images/spring/springcloud/hystrix/hystrix-server-dashboard.png)