---
title: eureka服务中心实例信息
tags: [spring]
---

1）修改注册到服务中心的服务名称

主要针对microservicecloud-provider-dept-8001模块（微服务提供者）

修改前服务列表的显示名称，其中status字段显示的实例名称为主机名+微服务名+端口号。显示不是很友好，我们可以在yml配置文件中自定义显示实例名称。

修改配置文件，即在eureka下添加instance配置信息

```
eureka:
 client: # 客户端注册eureka服务列表内
  service-url:
   defaultZone: http://localhost:7001/eureka
 instance: # 自定义注册到服务中心的实例名称
  instance-id: microservicecloud-dept8001
```

![](/images/spring/springcloud/eureka/eureka-server-provider-instance.png)

2）修改连接显示信息

主要针对microservicecloud-provider-dept-8001模块（微服务提供者）

当鼠标放在服务列表的实例上，浏览器的左下角显示的是localhost等连接信息，希望能够显示ip地址（更直观的信息）。

修改配置文件，在eureka:instance下添加prefer-ip-address属性

```
eureka:
 client: # 客户端注册eureka服务列表内
  service-url:
   defaultZone: http://localhost:7001/eureka
 instance: # 自定义注册到服务中心的实例名称
  instance-id: microservicecloud-dept8001
  prefer-ip-address: true
```

![](/images/spring/springcloud/eureka/eureka-server-provider-instance-link.png)

3）点击服务实例连接显示errorpage

修改microservicecloud-provider-dept-8001模块（微服务提供者）的pom文件，添加依赖

```
<!-- 监控信息 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

修改父工程springclouddemo，添加build，使系统能够读取到resouces下的配置信息

```
<build>
    <finalName>microservicecloud</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <!-- 针对resources目录下的资源文件，以$开头和结尾的作文变量引用 -->
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>
                    <delimit>$</delimit>
                </delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```

修改microservicecloud-provider-dept-8001模块（微服务提供者）的yml文件，添加info配置信息，其中以$引起来的数据都会当做变量来处理。

```
info:
 app.name: dept-microservicecloud
 company.name: www.test.com
 build.artifactId: $project.artifactId$
 build.version: $project.version$
```

![](/images/spring/springcloud/eureka/eureka-server-provider-instance-info.png)