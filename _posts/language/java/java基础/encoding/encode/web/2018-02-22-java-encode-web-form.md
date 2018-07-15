---
title: 浏览器form提交编码问题
tags: [coding]
---

form表单提交参数使用UTF-8编码，但并没有在ContentType中指定编码方式，因此服务器端仍然使用iso8859-1解码。

文本框中输入的值，在js中的编码是由页面的编码决定的（Content-Type中指定的编码）。后台解码后获取的字节数组，调用new-String获取对象时指定的编码要与页面编码相同才不会出现乱码的情况。

注：要求后台和前台页面使用的编码要统一设置。

1）页面

```
<form action="UrlEncodeServlet" method="post">
    <input type="hidden" name="hello" value="你好">
    <input type="submit" value="提交"></input>
</form>
```

2）使用火狐浏览器访问

![](/images/other/encode/firefox-form-encode.png)

服务器端--中文乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);

/# 输出
E4BDA0E5A5BD
ä½ å¥½
```

注：可以看出浏览器端提交参数使用的是UTF-8编码，服务器端没有正确解码，原因是form提交时，header头信息中添加的contentType没有指定编码，因此服务器端不知道该用什么编码方式进行解码，因此使用iso8859-1进行了解码。

```
Content-Type:"application/x-www-form-urlencoded"
```

3）解决方法

方案一：使用spring mvc编码过滤器，CharacterEncodingFilter。

该过滤器主要是解决form提交表单指定中文编码为UTF-8，只能解决post提交，不能改变get的解码方式（get是对URI进行编码的，只能通过服务器端设置URI编码实现）。

方案二：手动设置

```
# 设置读取数据的编码
request.setCharacterEncoding("UTF-8");
# 设置返回数据的编码
response.setCharacterEncoding("UTF-8")
```