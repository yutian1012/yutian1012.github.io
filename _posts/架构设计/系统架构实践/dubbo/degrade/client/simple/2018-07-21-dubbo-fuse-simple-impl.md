---
title: 使用dubbo搭建微服务架构-服务降级简单实现
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

2）启动测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务提供方法实现注册（运行DubboServiceApplication类）

最后启动调用端的测试代码（运行InvokeServiceTest类）。

测试结果：直接运行发现可以正常调用到服务，并没有返回null。

注：测试条能否变成绿色是查看是否能够正常执行熔断的判断

