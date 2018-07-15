---
title: java在开发环境中设置编码
tags: [coding]
---

1）IDE环境中设置java源文件编码字符集

eclipse环境中设置Text file encoding编码，是给Java编译器用的。

2）servlet从request中获取数据的编码

```
request.setCharacterEncoding("UTF-8");
```

servlet在准备从request中拿数据之前，会根据request.setCharacterEncoding设置的编码把request的数据转换过来。

*注：这种方式不能有效的解决URL中传递参数的中文编码问题，只能解决post方式提交的参数中文编码问题。*

3）servlet输入到response中的数据编码

```
response.setCharacterEncoding("UTF-8")
```

servlet准备输出数据前，会根据response.setCharacterEncoding设置的编码转换数据。

*注：这种方式设置的编码只会将输出的数据使用UTF-8进行编码，浏览器并不能知道返回数据使用的编码，因此有时会在浏览器端显示时出现乱码*

4）告知浏览器返回数据的编码

```
response.setContentType("text/html;charset=UTF-8");
```