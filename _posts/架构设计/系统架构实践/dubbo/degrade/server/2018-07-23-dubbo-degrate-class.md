---
title: 使用dubbo搭建微服务架构-服务降级实现类
tags: [architecture]
---

1）创建mock类

mock可以指定为一个具体的类，当调用失败时执行，以满足复杂的业务逻辑。mock类需要实现服务接口，类名规范为：接口名+Mock。

在dubbo-api-demo模块下创建IHelloMock类：

```
package org.sjq.dubbo.api.mock;

import org.sjq.dubbo.api.IHello;
/**
 * 服务降级处理类
 */
public class IHelloMock implements IHello{

    @Override
    public String say(String msg) {
        return null;
    }

}
```

注：可将mock类定义在公共API中

2）修改服务的提供端代码

修改dubbo-service-demo模块，IHello的实现类代码，在@Service注解上指定mock类

```
package org.sjq.dubbo.service;

import org.sjq.dubbo.api.IHello;

import com.alibaba.dubbo.config.annotation.Service;

@Service(loadbalance="roundrobin",mock="org.sjq.dubbo.api.mock.IHelloMock")
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

3）修改客户端引用

修改dubbo-client-demo模块InvokeService类，在@Reference注解上设置mock为true

```
package org.sjq.dubbo.client;

import org.sjq.dubbo.api.IHello;
import org.springframework.stereotype.Component;

import com.alibaba.dubbo.config.annotation.Reference;

@Component
public class InvokeService {
    @Reference(mock="true")
    public IHello hello;
}
```

注：经下面的测试，发现mock出现在一段即可实现服务降级

4）测试

首先前端zookeeper（/bin/zkServer.cmd）

启动微服务提供方法实现注册（运行DubboServiceApplication类）

最后启动调用端的测试代码（运行InvokeServiceTest类）。

测试结果：抛出异常ClassNotFoundException

注：测试条能否变成绿色是查看是否能够正常执行熔断的判断

5）错误

```
Caused by: java.lang.ClassNotFoundException: org.sjq.dubbo.api.IHelloMock
```

通过异常信息可以看出，查找的类路径与我们定义的类路径是不同的，我们定义的类路径为org.sjq.dubbo.api.mock.IHelloMock，而这里所找的是与接口在同一包下的类文件，该文件是不存在的。

解决方法：

方式一：将org.sjq.dubbo.api.mock.IHelloMock类移动到IHello同一包中。

方式二：修改注解@Reference(mock="true org.sjq.dubbo.api.mock.IHelloMock")

方式三：去掉@Reference注解的mock属性，只在服务器端配置，@Reference(timeout=2000)