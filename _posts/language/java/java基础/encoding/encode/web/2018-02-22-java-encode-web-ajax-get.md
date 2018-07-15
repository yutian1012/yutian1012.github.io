---
title: 浏览器页面使用ajax提交数据--get方式
tags: [coding]
---

js中的encodeURI和encodeURIComponent能对中文等字符进行UTF-8编码，然后传递到后台服务器。

### AJAX方式发送的GET请求对中文参数的影响

ajax的get方式在不同浏览器中对参数的编码方式不同。

1）js代码

```
<script type="text/javascript">
    $(function(){
        //点击事件
        $("#ajaxrequest").click(function(){
            var url="http://localhost:8080/java_web/UrlEncodeServlet?hello=你好";
            $.ajax({type:"GET", url:url});
        });
    })
</script>
```

页面链接访问

```
<a id="ajaxrequest" href="#">ajax提交数据</a>
```

2）使用firefox访问

![](/images/other/encode/firefox-url-ajaxget-query-encode.png)

注：可以看出firefox使用ajax发送的url对中文参数使用的是utf-8编码。

servlet获取请求数据

```
String hello=request.getParameter("hello");
//能正常输出中文字符，也能说明firefox使用了utf-8进行的编码
System.out.println(new String(hello.getBytes("iso8859-1"),"utf-8"));
```

3）IE浏览器访问

IE浏览器无法访问，服务器端会抛出异常。

解决方法：使用js的encodeURIComponent方法对中文进行转移，然后再提交。

```
var url="http://localhost:8080/java_web/UrlEncodeServlet?hello="+encodeURIComponent("你好");
```

4）总结

由于不同的操作系统、不同的浏览器、不同的网页字符集，将导致完全不同的编码结果。使用Javascript先对URL编码，然后再向服务器提交，不要给浏览器插手的机会。因为Javascript的输出总是一致的，所以就保证了服务器得到的数据是格式统一的。

方式一：encodeURI() 函数可对URL进行编码，

```
var url=encodeURI("http://localhost:8080/java_web/UrlEncodeServlet?hello=你好");
alert(url);
```

注：编码后，它输出符号的utf-8形式，并且在每个字节前加上%。对应的解码函数是decodeURI()。并且不对单引号进行编码。

方式二：encodeURIComponent()函数对应URL组成部分进行编码

```
var url="http://localhost:8080/java_web/UrlEncodeServlet?hello="+encodeURIComponent("你好");
```

注：对应的解码函数是decodeURIComponent()

5）encodeURIComponent与encodeURI()的区别

encodeURIComponent用于对URL的组成部分进行个别编码，而不用于对整个URL进行编码。并且encodeURIComponent能对某些特殊的字符进行编码。

```
# 下面的特殊字符在encodeURI()中不被编码的符号，
# 但在encodeURIComponent()中统统会被编码
; / ? : @ & = + $ , #
```

注：如果需要对特殊的字符进行编码就采用encodeURIComponent来实现。