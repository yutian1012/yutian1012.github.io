---
title: java在web环境中设置编码
tags: [coding]
---

Text file encoding 是给 Java 编译器用的。
2、request.setCharacterEncoding 是 servlet 在准备从 request 中拿数据之前设定的，API 会根据这个把 request 的数据转换过来。
3、response.setCharacterEncoding 是准备输出数据前设定的，字符将会被转换并且字符集声明出现在 Http Header 中，我们还应该在 <meta> 标签中再声明一次字符集，因为 Http Header 中的内容只在浏览器还连接在服务器上时有效，当我们保存网页之后再打开就只能靠 <meta> 标签来提示字符集了。