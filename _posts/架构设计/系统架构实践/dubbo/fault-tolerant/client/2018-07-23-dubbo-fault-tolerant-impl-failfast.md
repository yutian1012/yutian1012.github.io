---
title: 使用dubbo搭建微服务架构-集群容错策略-实战
tags: [architecture]
---

1）修改客户端调用

修改dubbo-client-demo模块的@Reference注解

```
package org.sjq.dubbo.client;

import org.sjq.dubbo.api.IHello;
import org.springframework.stereotype.Component;

import com.alibaba.dubbo.config.annotation.Reference;

@Component
public class InvokeService {
    @Reference(cluster="failfast",timeout=2000)
    public IHello hello;
}
```

2）修改服务提供者类

修改dubbo-service-demo模块的实现类HelloImpl，执行超时代码

```
package org.sjq.dubbo.service;

import org.sjq.dubbo.api.IHello;

import com.alibaba.dubbo.config.annotation.Service;

@Service
public class HelloImpl implements IHello{

    @Override
    public String say(String msg) {
        System.out.println("hello:"+msg);
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "provider"+msg;
    }
}
```

3）测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务提供方法实现注册（运行DubboServiceApplication类），启动2次，即2个服务提供者服务作为集群。

最后启动调用端的测试代码（运行InvokeServiceTest类）。

通过观察2个服务提供者的控制台输出，可以查看到只有一个服务都输出了相应的字符串。

4）抛出错误信息

```
com.alibaba.dubbo.rpc.RpcException: Failfast invoke providers dubbo://192.168.201.1:20881/org.sjq.dubbo.api.IHello?anyhost=true&application=consumer&check=false&cluster=failfast&dubbo=2.5.3&interface=org.sjq.dubbo.api.IHello&methods=say&pid=1416&side=consumer&timeout=2000&timestamp=1532352223171 RandomLoadBalance select from all providers [com.alibaba.dubbo.registry.integration.RegistryDirectory$InvokerDelegete@6cc0bcf6, com.alibaba.dubbo.registry.integration.RegistryDirectory$InvokerDelegete@29539e36] for service org.sjq.dubbo.api.IHello method say on consumer 192.168.201.1 use dubbo version 2.5.3, but no luck to perform the invocation. Last error is: Invoke remote method timeout.
```

注：这种方式只调用了一次服务，并且调用失败后快速抛出了异常。