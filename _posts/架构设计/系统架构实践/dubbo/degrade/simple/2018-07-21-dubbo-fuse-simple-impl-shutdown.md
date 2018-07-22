---
title: 使用dubbo搭建微服务架构-服务降级简单实现（关闭）
tags: [architecture]
---

1）只修改客户端访问

修改dubbo-client-demo项目的InvokeService类

```
package org.sjq.dubbo.client;

import org.sjq.dubbo.api.IHello;
import org.springframework.stereotype.Component;

import com.alibaba.dubbo.config.annotation.Reference;

@Component
public class InvokeService {
    @Reference(mock="return null")
    public IHello hello;
}
```

注：在@Referenc注解上配置mock属性，返回null

2）关闭服务提供者

将所有的服务提供者都关闭

3）启动测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务提供方法实现注册（运行DubboServiceApplication类）

最后启动调用端的测试代码（运行InvokeServiceTest类）。

测试失败

```
 No provider available for the service org.sjq.dubbo.api.IHello from the url zookeeper://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?application=consumer&dubbo=2.5.3&interface=org.sjq.dubbo.api.IHello&methods=say&mock=return+%E6%9C%8D%E5%8A%A1%E5%85%B3%E9%97%AD&pid=6264&side=consumer&timestamp=1532257263595 to the consumer 192.168.201.1 use dubbo version 2.5.3
```