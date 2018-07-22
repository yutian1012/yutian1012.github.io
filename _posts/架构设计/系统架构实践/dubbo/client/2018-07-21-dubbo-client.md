---
title: 使用dubbo搭建微服务架构-服务消费方
tags: [architecture]
---

1）创建maven module，并引入相关的依赖

参考dubbo-maven文章

2）编写远程调用dubbo服务

```
package org.sjq.dubbo.client;

import org.sjq.dubbo.api.IHello;
import org.springframework.stereotype.Component;

import com.alibaba.dubbo.config.annotation.Reference;

@Component
public class InvokeService {
    @Reference
    public IHello hello;
}
```

注：Dubbo的@Reference注解可以用于生成远程服务代理，@Reference提供了相关的配置属性

3）编写spring boot服务启动类

参考dubbo-maven文章

4）编写测试类

参考dubbo-maven文章

5）配置application.yml文件

参考dubbo-maven文章