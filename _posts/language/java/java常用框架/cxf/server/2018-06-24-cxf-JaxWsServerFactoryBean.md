---
title: CXF以JaxWsServerFactoryBean方式发布ws服务
tags: [webservice]
---

JaxWsServerFactoryBean是ServerFactoryBean的子类，因此，使用时与ServerFactoryBean非常类似。同时支持注解（可以通过注解修改wsdl文件），如使用@WebService修改服务的相关名词。

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
package org.sjq.test.cxfdemo.jaxswfactory;

import javax.jws.WebService;

import org.sjq.test.cxfdemo.factory.IHelloService;

@WebService(
    name="",//默认为服务类名，即JaxWsHelloServiceImpl
    serviceName="",//默认是服务类名+Service，即JaxWsHelloServiceImplService
    targetNamespace="",//默认是包名倒写
    portName=""//默认为类名+Port，即JaxWsHelloServiceImplPort
)
public class JaxWsHelloServiceImpl implements IHelloService{

    @Override
    public String sayHello(String name) {
        return "服务端jaxws：hello"+name;
    }

}
```

注：这里通过@WebService注解配置服务的相关信息

3）发布ws服务

```
package org.sjq.test.cxfdemo.jaxswfactory;

import org.apache.cxf.jaxws.JaxWsServerFactoryBean;

public class JaxWsCXFWebServicePublish {

    public static void main(String[] args){
        String url="http://localhost:7777/hello/jaxws";
        //获取CXF发表的服务类，
        JaxWsServerFactoryBean bean=new JaxWsServerFactoryBean();
        //设置发布路径
        bean.setAddress(url);
        //设置实现类
        bean.setServiceBean(new JaxWsHelloServiceImpl());
        //发表
        bean.create();
    }
}
```

4）浏览器查看wsdl

```
http://localhost:7777/hello/jaxws?wsdl
```
