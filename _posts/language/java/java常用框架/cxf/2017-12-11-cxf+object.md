---
title: CXF客户端向服务器发送对象
tags: [webservice]
---

1）使用soapui发送数据报错

```
<faultcode>soap:Server</faultcode>
<faultstring>Fault occurred while processing.</faultstring>
```

原因：查看服务器端实体类的hashcode需要的类属性值。该属性值必须传递，否则就出现异常。

解决方法：将服务器端类的hashcode方法中出现的属性，都必须在客户端传递时传递过去。

实例：

服务器端提供了一个addUser方法，传递User对象。而user对象的hashcode方法中，使用account字段来计算hashcode值，因此客户端传递数据时必须要为account属性赋值。

```
@Override
public int hashCode() {
    return this.account.hashCode();
}
```

客户端发送到soap消息

```
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <ns2:addUser xmlns:ns2="http://xxxx.com/">
            <user>
                <account>admin</account>
                ...
            </user>
        </ns2:addUser>
    </soap:Body>
</soap:Envelope>
```

2）Unmarshalling错误

将字符串转换成java对象，称为Unmarshalling。

```
<faultcode>soap:Client</faultcode>
<faultstring>Unmarshalling Error:</faultstring>
```

客户端发送信息

```
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <ns2:addUser xmlns:ns2="http://xxxx.com/">
            <user>
                <account>admin</account>
                <createBy></createBy>
                ...
            </user>
        </ns2:addUser>
    </soap:Body>
</soap:Envelope>
```

注：原因是createBy标签中没有传递任何值，如果不传递值，必须去掉，否则就会出现Unmarshalling错误。