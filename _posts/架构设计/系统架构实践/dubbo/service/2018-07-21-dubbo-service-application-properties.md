---
title: 使用dubbo搭建微服务架构-服务提供者application配置
tags: [architecture]
---

在创建的spring boot项目下，可在配置文件application.properties或application.yml中配置dubbo的相关属性。

spring.dubbo.scan用于配置扫描服务所在包路径。

常见配置项

1）与应用相关的配置，spring.dubbo.application开头的配置

spring.dubbo.application.name

当前应用名称，用于注册中心计算应用间依赖关系。

spring.dubbo.application.version

当前应用的版本号

spring.dubbo.application.logger

日志输出方式，可选：slf4j，jcl，log4j，jdk

2）与注册中心相关配置，spring.dubbo.registry开头的配置

spring.dubbo.registry.address

服务所使用的注册中心，A://B:C（如，zookeeper://127.0.0.1:2181），其中A为注册中心类型（取值为：zookeepper,multicast,redis,simple），B为注册中心所在ip地址，C为注册中心的端口号。

注：如果配置集群的时候使用逗号分隔多个注册中心，如A://B1:C1,A://B2:C2

spring.dubbo.registry.username

登录注册中心用户名，如果注册中心不需要验证可不填

spring.dubbo.registry.password

登录注册中心密码，如果注册中心不需要验证可不填

spring.dubbo.registry.timeout

注册中心请求超时时间（毫秒）

spring.dubbo.registry.register

是否向注册中心注册服务，如果设为false，将只订阅，不注册。

spring.dubbo.registry.subscribe

是否向注册中心订阅服务，如果设为false，将只注册，不订阅。

spring.dubbo.registry.transporter

网络传输方式，可选mina，netty

3）协议相关的配置，spring.dubbo.protocol开头的配置

spring.dubbo.protocol.name

服务所使用的协议名称

spring.dubbo.protocol.port

服务提供方所暴露的端口号，多个服务提供方不可重复。

注：dubbo协议缺省端口为20880，RMI协议缺省端口为1099，HTTP和Hessian协议缺省为80。如果配置为-1或没有配置，则会分配一个没有被占用的端口。

spring.dubbo.protocol.threadpool

线程池类型，可选：fixed，cached

spring.dubbo.protocol.threads

服务线程池大小（固定大小）

spring.dubbo.protocol.iothreads

IO线程池大小（固定大小）

spring.dubbo.protocol.accepts

服务提供方最大可接受连接数

spring.dubbo.protocol.payload

请求及响应数据包大小限制，单位为字节，默认为8838860B（8MB）

spring.dubbo.protocol.serialization

协议序列化方式，当协议支持多种序列化方式时使用，如dubbo，hession2，java原生，以及http协议的json等。