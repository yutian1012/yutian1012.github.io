---
title: swagger最佳实践
tags: [spring]
---

方式一：

返回对象一般使用json包装，json中包含了状态信息

```
package com.example.demo.entity;

import java.util.Map;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Response {
    /** 返回信息码*/
    private String rspCode="000000";
    /** 返回信息内容*/
    private String rspMsg="操作成功";
    
    private String url;//跳转连接
    
    private Map<String,Object> data;//返回数据信息
    ...
}
```

注：这种方式返回的实体对象中包含了状态码，使用ajax获取数据后可以通过状态码进行判断。

问题：全局异常的处理？

方式二：使用http状态标识信息

controller方法直接返回对应的实体类，而状态码通过http协议状态来指定。

问题：ajax如何获取异常信息，js前端的try...catch能处理吗？待验证