---
title: web中字符的编码与解码
tags: [coding]
---

web协议支持中文的解决方案

### 通过ISO编码转换

1）web前端将中文使用iso转成字符串

有时候，为了让中文字符适应某些特殊要求（如http header头要求其内容必须为iso8859-1编码），可能会通过将中文字符按照字节方式来编码的情况。如 

```
String s_iso88591 = new String("深".getBytes("UTF-8"),"ISO8859-1")， 
```

2）服务器端解析

，在将这些字符传递到目的地后，目的地程序再通过相反的方式处理，来得到正确的中文汉字"深"。这样就既保证了遵守协议规定、也支持中文。 

```
String s_utf8 = new String(s_iso88591.getBytes("ISO8859-1"),"UTF-8")
```
