---
title: SimpleCaptcha验证码
tags: [web]
---

参考：https://www.cnblogs.com/panxuejun/p/7623840.html

1）url特殊字符串异常

```
java.lang.IllegalArgumentException: Illegal character in query...
```

原因，url访问路径中存在特殊字符，如空格等，需要在url中被替换成%20


解决方式

```
URL url = new URL(strUrl);

URI uri = new URI(url.getProtocol(), url.getHost(), url.getPath(), url.getQuery(), null);

HttpClient client    = new DefaultHttpClient();
HttpGet httpget = new HttpGet(uri);
// 或者
HttpGet httpget = new HttpGet(uri.toString);
```

注：使用uri将特殊字符过滤掉