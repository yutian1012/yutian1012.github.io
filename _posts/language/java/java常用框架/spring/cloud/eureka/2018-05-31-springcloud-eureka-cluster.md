---
title: spring cloud eureka集群
tags: [spring]
---

目前只有一个服务注册中心microservicecloud-eureka-7001，一旦该服务宕机，会造成整个系统的服务无法访问，因此，这里要配置成集群，实现高可用。

1）新建eureka集群

创建模块microservicecloud-eureka-7002和microservicecloud-eureka-7003，使之成为一个集群对外提供服务。模块的创建参考microservicecloud-eureka-7001

a.模块microservicecloud-eureka-7002的pom信息

```
  <artifactId>microservicecloud-eureka-7002</artifactId>
  
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

b.模块microservicecloud-eureka-7003的pom信息

```
  <artifactId>microservicecloud-eureka-7003</artifactId>
  
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

复制microservicecloud-eureka-7001模块的配置文件application.yml，到相应模块的resources目录下，并修改port。（这里简化只显示port信息，其他配置信息进行了简化）

c.模块microservicecloud-eureka-7002的yml信息

```
server:
 port: 7002
```

d.模块microservicecloud-eureka-7003的yml信息

```
server:
 port: 7003
```

复制microservicecloud-eureka-7001模块的启动类EurekaServer7001_App.

e.模块microservicecloud-eureka-7002中启动类命名为EurekaServer7002_App

f.模块microservicecloud-eureka-7003中启动类命名为EurekaServer7003_App

2）修改eureka服务器名称

microservicecloud-eureka-7001中的服务器名称配置如下这里的名称为localhost，为了区分不同的eureka集群，需要为集群中的eureka服务器重新命名。（这里简化只显示hostname信息，其他配置信息进行了简化）

```
eureka:
 instance:
  hostname: localhost # eureka服务端的实例名称
```

修改C:\Windows\System32\drivers\etc\hosts文件，同时修改配置文件中eureka的服务器名称与之对应.

```
127.0.0.1   eureka7001.com
127.0.0.1   eureka7002.com
127.0.0.1   eureka7003.com
```

a.microservicecloud-eureka-7001修改为

```
eureka:
 instance:
  hostname: eureka7001.com # eureka服务端的实例名称
```

b.microservicecloud-eureka-7002修改为

```
eureka:
 instance:
  hostname: eureka7002.com # eureka服务端的实例名称
```

c.microservicecloud-eureka-7003修改为

```
eureka:
 instance:
  hostname: eureka7003.com # eureka服务端的实例名称
```

3）创建eureka集群间关联

目前eureka之间谁都不认识谁，如何创建关联呢。

修改各自的yml配置文件，通过eureka:service-url:defaultZone属性创建关联（这里简化只显示defaultZone信息，其他配置信息进行了简化）

a.microservicecloud-eureka-7001修改为

```
eureka:
 client:
  service-url:
   defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

注：配置其他以外的eureka服务地址

b.microservicecloud-eureka-7002修改为

```
eureka:
 client:
  service-url:
   defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/
```

c.microservicecloud-eureka-7003修改为

```
eureka:
 client:
  service-url:
   defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

4）修改服务提供者microservicecloud-provider-dept-8001模块

让该模块同时注入到eureka的3个服务注册中心，修改microservicecloud-provider-dept-8001模块的yml配置文件

```
eureka:
 client: # 客户端注册eureka服务列表内
  service-url:
   #defaultZone: http://localhost:7001/eureka
   defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

注：这里配置defaultZone同时指向3个eureka服务注册中心，即向3个注册中心同时注册

5）启动测试

先启动3个eureka服务，再启动provider服务，访问注册中心查看详细信息。