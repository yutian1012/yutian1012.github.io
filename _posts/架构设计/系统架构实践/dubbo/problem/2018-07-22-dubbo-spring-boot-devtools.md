---
title: 使用dubbo搭建微服务架构-常见问题
tags: [architecture]
---

在网关服务中添加了spring-boot-devtools依赖，启动微服务错误信息

```
Failed to init remote service reference at filed hello in class org.sjq.dubbo.gateway.RpcController, cause: interface org.sjq.dubbo.api.IHello is not visible from class loader, dubbo version: 2.5.3, current host: 192.168.201.1

java.lang.IllegalArgumentException: interface org.sjq.dubbo.api.IHello is not visible from class loader
```

解决办法，去掉该依赖。