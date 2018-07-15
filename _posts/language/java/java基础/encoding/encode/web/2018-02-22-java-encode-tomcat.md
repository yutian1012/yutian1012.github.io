---
title: tomcat设置编码
tags: [coding]
---

1）在tomcat的server.xml设置URIEncoding

```
# 指定使用utf8对URI中出现的中文进行decode
<Connector URIEncoding="utf-8"  />
# 浏览器地址栏中输入
http://localhost:8080/test/测试.do
# 访问时浏览器自动编码，浏览器一般使用UTF-8进行编码
http://localhost:8080/test/%E6%B5%8B%E8%AF%95.do
```

通过上面的设置，服务器端tomcat会对编码进行解码处理，使用的时UTF-8，与客户端保持一致，这样在获取参数或URIInfo时就不会出现乱码问题了。

2）在tomcat的server.xml设置useBodyEncodingForURI

```
# 使用http header中指定charset进行decode
# 会到请求头中查找Content-Type，使用Content-Type指定的编码进行URI解码
<Connector useBodyEncodingForURI="true"  />
```

注：使用默认值ISO-8859-1