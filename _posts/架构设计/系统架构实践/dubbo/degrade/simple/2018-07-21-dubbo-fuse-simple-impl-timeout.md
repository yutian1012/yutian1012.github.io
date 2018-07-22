---
title: 使用dubbo搭建微服务架构-服务降级简单实现（超时）
tags: [architecture]
---

1）修改dubbo-client-demo项目（服务调用者）的InvokeService类

```
@Component
public class InvokeService {
    @Reference(mock="return null",timeout=2000,retries=0)
    public IHello hello;
}
```

注：设置了timeout属性值为2s超时，并且不需要重复尝试调用

2）修改dubbo-service-demo项目（服务提供者）的HelloImpl类

修改服务提供者项目（通过超时返回熔断信息），通过让调用超时实现熔断处理，在服务端调用sleep方法休眠，同时客户端配置超时时间

```
package org.sjq.dubbo.service;

import org.sjq.dubbo.api.IHello;

import com.alibaba.dubbo.config.annotation.Service;

@Service(loadbalance="roundrobin")
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

注：设置服务提供者休眠3秒，超过客户端调用超时时间2秒

3）运行测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务提供方法实现注册（运行DubboServiceApplication类）

最后启动调用端的测试代码（运行InvokeServiceTest类）。

测试结果：直接运行发现返回null。