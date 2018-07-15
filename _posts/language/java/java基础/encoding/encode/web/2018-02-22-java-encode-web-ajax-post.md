---
title: 浏览器页面使用ajax提交数据--post方式
tags: [coding]
---

浏览器中ajax以post方式发送请求数据，都是使用UTF-8进行编码的。并且会在请求头中设置Content-Type编码为UTF-8

```
Content-Type:"application/x-www-form-urlencoded; charset=UTF-8"
```

servlet端获取request对象的Content-Type

```
System.out.println(request.getContentType());
```

注：通过这里的输出也能够验证浏览器端传递的Content-Type值。

### AJAX方式发送的POST请求对中文参数的影响

1）js代码

```
<script type="text/javascript">
    $(function(){
        $("#ajaxpost").click(function(){
            var url="http://localhost:8080/java_web/UrlEncodeServlet";
            $.post(url,{"hello":"你好"});
        });
    })
</script>
```

注：$.post提交时需要注意，参数必须使用双引号括起来（不能使用单引号），否则无法执行。

2）使用firefox访问

![](/images/other/encode/firefox-url-ajaxpost-query-encode.png)

注：firefox以post方式提交的数据中文使用utf-8编码。

servlet获取请求数据

```
String hello=request.getParameter("hello");
//能正常输出中文字符，也能说明firefox使用了utf-8进行的编码
System.out.println(new String(hello.getBytes("iso8859-1"),"utf-8"));
```

3）使用IE浏览器访问

![](/images/other/encode/ie-url-ajaxpost-query-encode.png)

注：IE浏览器的post提交中文参数也是使用utf-8进行编码的

4）结论：

ajax的post提交对中文参数使用UTF-8编码，并能在请求头的ContentType中指定编码方式，因此服务器端能够使用正确的编码方式进行解码。

5）服务器端正确获取post数据

方式一：

在获取post提交的数据前需要先调用request.setCharacterEncoding方法，设置编码为UTF-8，然后在调用request.getParameter就能正确获取数据，而不会显示乱码了。

```
request.setCharacterEncoding("UTF-8");
String hello=request.getParameter("hello");
System.out.println(hello);
```

方式二：

无需调用setCharacterEncoding方法，因为post会自动设置Content-Type编码为UTF-8，request在获取时会自动根据Content-Type中的编码获取数据。