---
title: webservice服务使用编码方式实现客户端调用
tags: [webservice]
---

使用javax.xml.ws.Service类用于访问web服务。在客户端需要一个与服务器接口完全相同的类。

### 1. 客户端代码编写

```
package org.ipph.web.webservice.client;

import java.net.URL;

import javax.xml.namespace.QName;
import javax.xml.ws.Service;

import org.ipph.web.webservice.IWebService;

public class WSClientCoding {
    public static void main(String[] args) throws Exception {
        Service service =Service.create(new URL("http://10.33.6.199:8989/WS_Server/Webservice?wsdl"), new QName("http://webservice.web.ipph.org/","WebServiceImplService"));

        IWebService webService=service.getPort(new QName("http://webservice.web.ipph.org/", "WebServiceImplPort"),IWebService.class);
        
        String resResult = webService.sayHello("孤傲苍狼");
        System.out.println("调用WebService的sayHello方法返回的结果是："+resResult);
        System.out.println("---------------------------------------------------");
        //调用WebService的save方法
        resResult = webService.save("孤傲苍狼","123");
        System.out.println("调用WebService的save方法返回的结果是："+resResult);
    }
}
```

注：IWebService是与服务器的相同，当然这里也可以使用wsimport工具生成的WebServiceImpl接口。

### 2. 客户端编写的参数

1）Service类的create方法

```
Service javax.xml.ws.Service.create(
    URL wsdlDocumentLocation, QName serviceName)

wsdlDocumentLocation: URL for the WSDL document location for the service
serviceName: QName for the service
```

注：serviceName需要targetNamespace和serviceName，参考wsdl的definitions和service节点。

![](/images/java_structure/webservice/service-create-ns.png)

![](/images/java_structure/webservice/service-create-servername.png)

2）Service类的getPort方法

```
<IWebService> IWebService javax.xml.ws.Service.getPort(
    QName portName, Class<IWebService> serviceEndpointInterface)

portName: Qualified name of the service endpoint in the WSDL service description.
serviceEndpointInterface: Service endpoint interface supported by the dynamic proxy instance.
```

注：portName需要targetNamespace和portName，参考wsdl的definitions和service节点。

![](/images/java_structure/webservice/service-create-ns.png)

![](/images/java_structure/webservice/service-create-portname.png)

注：关键类QName，被称为完全限定名（Qualified Name的缩写）。