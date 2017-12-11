---
title: webservice服务使用JDK方式实现客户端调用
tags: [webservice]
---

### 1. 生成客户端代码

借助jdk的wsimort.exe工具生成客户端代码，wsimort.exe工具位于Jdk的bin目录下。

1）执行命令：

wsimport -keep url(url为wsdl文件的路径)生成客户端代码

```
wsimport -keep -s D:/src -p org.ipph.web.webservice.client http://10.33.6.199:8989/WS_Server/Webservice?wsdl
```

![](/images/java_structure/webservice/webservicewsimportgencode.png)

命令执行完成后，会将生成的源码存放到D:/src目录下，可以将该源码放入到客户端的项目中，在客户端编写时调用该java类完成服务的调用。

注：wsimport.exe，这个工具依据wsdl文件生成相应的类文件，然后用这些类文件，就可以像调用本地的类一样调用WebService提供的方。

2）常用参数

-d:生成客户端执行类的class文件的存放目录

-s:生成客户端执行类的源文件的存放目录

-p:定义生成类的包名

-keep:保存生成的文件

### 2. 借助生成的代码编写客户端

```
package org.ipph.web.webservice.client;

public class WSClient {
    public static void main(String[] args) {
        //创建一个用于产生WebServiceImpl实例的工厂，WebServiceImplService类是wsimport工具生成的
        WebServiceImplService factory = new WebServiceImplService();
        //通过工厂生成一个WebServiceImpl实例，WebServiceImpl是wsimport工具生成的
        WebServiceImpl wsImpl = factory.getWebServiceImplPort();
        //调用WebService的sayHello方法
        String resResult = wsImpl.sayHello("孤傲苍狼");
        System.out.println("调用WebService的sayHello方法返回的结果是："+resResult);
        System.out.println("------------------------------------------");
        //调用WebService的save方法
        resResult = wsImpl.save("孤傲苍狼","123");
        System.out.println("调用WebService的save方法返回的结果是："+resResult);
    }
}
```

注：类中引入的所有类都是wsimport工具生成的。

### 3. 生成源码分析

![](/images/java_structure/webservice/wsimportgencodes.png)

