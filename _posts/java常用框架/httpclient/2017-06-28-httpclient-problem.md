---
title: httpclient常见错误
tags: [httpclient]
---

### URISyntaxException（URI语法异常）

错误提示

```
java.lang.IllegalArgumentException: Illegal character in path at index 52: http://192.168.203.178/webmodel/cpquery/isValidToken 
...
java.net.URISyntaxException: Illegal character in path at index 52:http://192.168.203.178/webmodel/cpquery/isValidToken 
```

注：这是属于URI地址错误，仔细观察，在http地址的52位存在一个空格（连接的末尾），去掉该空格就可以正常执行了。