---
title: spring cloud ribbon在多服务提供者间实现负载均衡
tags: [spring]
---

1）服务提供者

目前只有一个服务提供者，即microservicecloud-provider-dept-8001模块，ribbon每次访问都会访问到该服务提供者，并不能实现真正意义上的负载均衡，因为只有一个服务提供者。

下面新建另外2个服务提供者，分别为：microservicecloud-provider-dept-8002和microservicecloud-provider-dept-8003.

2）新建2个服务提供者模块，参考microservicecloud-provider-dept-8001模块

microservicecloud-provider-dept-8002模块

```
<parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springclouddemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>microservicecloud-provider-dept-8002</artifactId>
```

microservicecloud-provider-dept-8003模块

```
<parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springclouddemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>microservicecloud-provider-dept-8003</artifactId>
```

3）对每个模块添加依赖，依赖于8001模块一致

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

为每个模块配置不同的包名

```
# 模块8002
<build>
    <finalName>microservicecloudProviderDept8002</finalName>
</build>

# 模块8003
<build>
    <finalName>microservicecloudProviderDept8003</finalName>
</build>
```

4）拷贝代码，并修改相应的启动类名称（便于理解）

修改启动类名称

```
# 模块8002
@SpringBootApplication
@EnableEurekaClient //服务启动后自动注册到eureka服务中心
@EnableDiscoveryClient //启动服务发现功能
public class DeptProvider8002_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8002_App.class, args);
    }
}
# 模块8003
@SpringBootApplication
@EnableEurekaClient //服务启动后自动注册到eureka服务中心
@EnableDiscoveryClient //启动服务发现功能
public class DeptProvider8003_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8002_App.class, args);
    }
}
```

5）拷贝修改配置文件

修改端口号，修改数据库配置，修改实例名称，但一定不要修改服务名称。

a.模块8002

```
server:
 port: 8002

# 服务名一定不能变，并修改数据库的配置
spring:
 application:
  name: microservicecloud-dept 
 datasource:
  type: com.alibaba.druid.pool.DruidDataSource      # 当前数据源操作类型
  driver-class-name: org.gjt.mm.mysql.Driver        # mysql驱动包
  url: jdbc:mysql://localhost:3306/cloudDB02        #  数据库名称

# 修改配置到eureka的实例名
eureka:
 instance: # 自定义注册到服务中心的实例名称
  instance-id: microservicecloud-dept8002
```

b.创建相应的数据库

```
drop database if exists cloudDB02;
create database cloudDB02 character set UTF8;
use cloudDB02;

create table dept(
    deptno bigint not null primary key auto_increment,
    dname varchar(60),
    db_source varchar(60)
);

insert into dept(dname,db_source) values('开发部',database());
insert into dept(dname,db_source) values('人事部',database());
insert into dept(dname,db_source) values('财务部',database());
insert into dept(dname,db_source) values('市场部',database());
insert into dept(dname,db_source) values('运维部',database());
```

注：模块8003与之类似

6）启动并测试

```
# 进入上传目录
cd /root/tmp
# 启动eureka服务注册中心
nohup java -jar microservicecloudEureka7001.jar >/dev/null &
nohup java -jar microservicecloudEureka7002.jar >/dev/null &
nohup java -jar microservicecloudEureka7003.jar >/dev/null &
# 启动服务提供者
nohup java -jar microservicecloudProviderDept8001.jar >/dev/null &
nohup java -jar microservicecloudProviderDept8002.jar >/dev/null &
nohup java -jar microservicecloudProviderDept8003.jar >/dev/null &
# 启动服务消费者
nohup java -jar microservicecloudConsumerDept80.jar >/dev/null &
```

测试每一个服务提供者

```
# 提供者8001
http://47.104.19.xx:8001/dept/list
# 提供者8002
http://47.104.19.xx:8002/dept/list
# 提供者8003
http://47.104.19.xx:8003/dept/list
```

测试消费者，不断刷新页面，会发现数据源不断切换，即访问的服务不断变化，从而证明了以轮询的方式实现了负载均衡。

```
http://47.104.19.xx/consumer/dept/get/1
```

7）查看服务注册中心

![](/images/spring/springcloud/ribbon/ribbon-multi-provider.png)