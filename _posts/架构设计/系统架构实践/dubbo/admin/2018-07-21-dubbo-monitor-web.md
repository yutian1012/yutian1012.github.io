---
title: 使用dubbo搭建微服务架构-监控中心界面
tags: [architecture]
---

1）服务管理

启动服务提供者或服务消费者项目，便可在服务治理目录下的提供者和消费者页面中查看对应的项。

![](/images/architecture/application/dubbo/dubbo-admin/service-admin.png)

2）服务

服务提供方中所暴露的具体服务，与注解@Service相关

```
@Service
public class helloImpl implements IHello{
    @Override
    public String say(String msg) {
      ....
    }
}
```

![](/images/architecture/application/dubbo/dubbo-admin/dubbo-service.png)

3）应用

服务提供方或消费方的名称，有application.properties配置文件中的spring.dubbo.application.name参数指定。

```
# 消费方的配置
spring:
 dubbo:
  application:
   name: consumer
```

![](/images/architecture/application/dubbo/dubbo-admin/application-admin.png)

4）机器

服务提供方或消费方所运行的环境，有ip地址与端口号进行区分。