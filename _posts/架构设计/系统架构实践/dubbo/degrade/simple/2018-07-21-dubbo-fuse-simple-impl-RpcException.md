---
title: 使用dubbo搭建微服务架构-服务降级简单实现（异常）
tags: [architecture]
---

1）修改dubbo-client-demo项目（服务调用者）的InvokeService类

```
@Component
public class InvokeService {
    @Reference(mock="return null")
    public IHello hello;
}
```

2）修改dubbo-service-demo项目（服务提供者）的HelloImpl类

```
package org.sjq.dubbo.service;

import org.sjq.dubbo.api.IHello;

import com.alibaba.dubbo.config.annotation.Service;
import com.alibaba.dubbo.rpc.RpcException;

@Service(loadbalance="roundrobin")
public class HelloImpl implements IHello{

    @Override
    public String say(String msg) {
        System.out.println("hello:"+msg);
        throw new RpcException();
        //return "provider"+msg;
    }
}
```

注：这里通过抛出RpcException异常来测试熔断执行

3）运行测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务提供方法实现注册（运行DubboServiceApplication类）

最后启动调用端的测试代码（运行InvokeServiceTest类）。

测试失败，抛出RpcException异常，甚至抛出RuntimeException异常都无法返回null。