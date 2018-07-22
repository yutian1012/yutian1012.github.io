---
title: 使用dubbo搭建微服务架构-服务提供者
tags: [architecture]
---

1）创建maven module，并引入相关的依赖

参考dubbo-maven文章

2）创建接口实现类

```
package org.sjq.dubbo.service;

import org.sjq.dubbo.api.IHello;

import com.alibaba.dubbo.config.annotation.Service;

@Service
public class helloImpl implements IHello{

    @Override
    public String say(String msg) {
        System.out.println("hello:"+msg);
        return msg;
    }
}
```

注：dubbo的@Service注解用于暴露服务

3）创建服务启动类

参考dubbo-maven文章

4）配置application.yml文件

参考dubbo-maven文章