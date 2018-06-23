---
title: CXF以ServerFactoryBean方式发布ws服务
tags: [webservice]
---

ServerFactoryBean可以创建ws服务，通过绑定地址和端口，设置相应的服务类。将该服务类提供的方法作为ws服务访问，对外提供ws服务。

注：可通过main方法直接发表一个ws服务（可参考入门实例代码）

1）定义接口

```
package org.sjq.test.cxfdemo.factory;

public interface IHelloService {
    /**
     * 客户端传递name作为参数
     * @param name
     * @return
     */
    public String sayHello(String name);
}
```

2）实现类

```
package org.sjq.test.cxfdemo.factory;

public class HelloServiceImpl implements IHelloService {

    @Override
    public String sayHello(String name) {
        return "服务端：hello"+name;
    }

}
```

3）发布ws服务

```
package org.sjq.test.cxfdemo.factory;

import org.apache.cxf.frontend.ServerFactoryBean;

public class CXFWebServicePublish {
    
    public static void main(String[] args) {
        String url="http://localhost:7777/hello";
        //获取CXF发表的服务类，
        //不支持注解的方式，但是这样就不能通过注解来修改wsdl文件
        ServerFactoryBean bean=new ServerFactoryBean();
        //设置发布路径
        bean.setAddress(url);
        //设置实现类
        bean.setServiceBean(new HelloServiceImpl());
        //发表
        bean.create();
    }
}
```

4）浏览器查看wsdl

```
http://localhost:7777/hello?wsdl
```

5）设置ws相关的服务名

