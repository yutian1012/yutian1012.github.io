---
title: webservice安全认证头
tags: [webservice]
---

### 安全认证头信息

1）安全认证错误

有时候调用web service会出现A security error的错误，是因为webservice的服务端需要提供soap认证的表头。

```
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
   <soap:Body>
      <soap:Fault>
         <faultcode xmlns:ns1="http://ws.apache.org/wss4j">ns1:SecurityError</faultcode>
         <faultstring>A security error was encountered when verifying the message</faultstring>
      </soap:Fault>
   </soap:Body>
</soap:Envelope>
```

2）认证表头的信息和格式

在标签soap:Envelope和soap:Body之间会添加

```
<soapenv:Header>
    <wsse:Security soapenv:mustUnderstand="1" xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">
        <wsse:UsernameToken wsu:Id="UsernameToken-A471E661D4C93BAAED14944689305333">

            <wsse:Username>huwei</wsse:Username>

            <wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordText">hello</wsse:Password>

            <wsse:Nonce EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary">
            yvWY5cpVN4YSbqk/yd6fHA==</wsse:Nonce>

            <wsu:Created>2017-05-11T02:15:30.533Z</wsu:Created>

        </wsse:UsernameToken>
    </wsse:Security>
</soapenv:Header>
```

参考：http://blog.csdn.net/oscar999/article/details/40340819