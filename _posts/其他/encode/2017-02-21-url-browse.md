---
title: 浏览器URL编码
tags: [other]
---

URL编码问题，只有字母和数字[0-9a-zA-Z]、一些特殊符号$-_.+!*'(),[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。如果URL中有汉字，就必须编码后使用。

### 1. 创建一个servlet来模拟服务器端

1）创建UrlEncodeServlet类

```
public class UrlEncodeServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doService(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doService(request, response);
    }
    protected void doService(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String url=request.getPathInfo();//解码后的url信息
        System.out.println(request.getServletPath());
        System.out.println(url);
        System.out.println(request.getRequestURI());//未解码前
    }
}
```

2）测试

```
http://localhost:8080/java_web/UrlEncodeServlet/hello
-------------------
http://localhost:8080/java_web/UrlEncodeServlet/你好
```

输出内容为：

```
/UrlEncodeServlet ---ServletPath
/hello --pathInfo
/java_web/UrlEncodeServlet/hello
----------------
/UrlEncodeServlet
/你好
/java_web/UrlEncodeServlet/%E4%BD%A0%E5%A5%BD
```

### 2. 网址路径中包含汉字

1）访问路径：

```
http://localhost:8080/java_web/UrlEncodeServlet/你好
```

2）在火狐浏览器中运行，并使用firebug查看发送的内容

```
http://localhost:8080/java_web/UrlEncodeServlet/%E4%BD%A0%E5%A5%BD
```

![](/images/other/encode/firefox-url-encode1.png)

点击编辑和重发按钮，可以查看到实际发生的url地址（被编码了）

![](/images/other/encode/firefox-url-encode2.png)

3）在IE浏览器中运行，并捕获网络发送信息

IE浏览器可以设置发生URI时所使用的编码

![](/images/other/encode/ie-url-encode.png)

后台通过request.getRequestURI()也可以看到其编码后的uri值

```
/java_web/UrlEncodeServlet/%E4%BD%A0%E5%A5%BD
```

4）chome浏览器中运行

编码值为：

```
http://localhost:8080/java_web/UrlEncodeServlet/%E4%BD%A0%E5%A5%BD
```

![](/images/other/encode/chrome-url-encode.png)

5）后台"你好"字符串对应utf-8的编码字节

```
public static void main(String[] args) throws UnsupportedEncodingException {
    System.out.println(DatatypeConverter.printHexBinary("你好".getBytes("UTF-8")));
}
//输出E4BDA0E5A5BD
```

注：可以看到utf-8编码的"你好"与uri编码的值是相同的，从而也可以确定浏览器所使用的uri编码方式是utf-8

补充：如过服务器端出现的URI地址中汉字显示乱码，只能通过服务器设置uri的编码方式解决。如在tomcat服务器中，可以设置server.xml中的URIEncoding="UTF-8"，这样服务器端在获取浏览器发送的uri时，会认为浏览器使用的是utf-8对uri进行编码，因此使用相同的utf-8对其进行解码，因此在uri中的中文字符就不会出现乱码（保证两端的编码/解码方式相同）。

### 3. 查询字符串包含汉字

1）访问路径：

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=你好
```

注：目前汉字在查询参数中，而不在uri中，与上面的方式是不同的。

服务器端编码设置：

```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" />
```

服务器端显示接收的参数，并打印出获取的字节数组

```
protected void doService(HttpServletRequest request, 
    HttpServletResponse response) throws ServletException, IOException {
    String hello=request.getParameter("hello");
    System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
    System.out.println(hello);
}
```

2）火狐浏览器查看

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=%E4%BD%A0%E5%A5%BD
```

![](/images/other/encode/firefox-url-query-encode1.png)

服务器端的参数显示乱码。

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
ä½ å¥½
```

注：可以看出firefox对查询参数使用的编码是utf-8，而服务器端解码时使用的是iso8859-1，所以会显示乱码。

服务器端可以进行转码操作

```
System.out.println(new String(hello.getBytes("iso8859-1"),"utf-8"));
```

3）IE浏览器

ie浏览器的开发者工具无法查看到uri编码后的值，因此只能通过分析服务器端的字节数组，确定其查询参数使用的编码。

服务器端的输出，依然中文乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
C4E3BAC3
ÄãºÃ
```

注：可以明显的看出浏览器对查询参数使用的编码不再是utf-8，猜测是GBK或GB2312。

验证：

```
System.out.println(DatatypeConverter.printHexBinary("你好".getBytes("GBK")));
//输出值为C4E3BAC3
//或者通过new String方式验证
System.out.println(new String(hello.getBytes("iso8859-1"),"GBK"));
//正确的输出"你好"
```

注：通过服务器的转码，可以看出IE浏览器对查询字段使用的编码为GBK。

问题：header头的Content-Type指定的编码对浏览器GET的编码有何影响？

在jsp页面的header中添加

```
<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
```

使用IE浏览器中点击链接的方式发生get请求(jsp页面)

```
<a href="http://localhost:8080/java_web/UrlEncodeServlet?hello=你好">中文参数</a>
```

服务器端代码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出：
E4BDA0E5A5BD
ä½ å¥½
```

注：可以看出显示浏览器对查询参数的编码不再是使用GBK（上面得出的结论），而变成了UTF-8编码了。但是去掉meta再次点击依然是使用UTF-8编码。

```
<%@page contentType="text/html; charset=UTF-8" %>
```

原因，jsp页面中存在charset，该编码会在浏览器接收到响应内容时，使用此编码进行页面的解析渲染，因此页面中的中文连接也会使用UTF-8进行编码，传递到后台自然是使用UTF-8进行编码的值（解析网页的编码）。

结论：meta指定charset不会对页面的get产生影响，而应该与解析网页的编码有关。

4）chrome浏览器

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=%E4%BD%A0%E5%A5%BD
```

![](/images/other/encode/chrome-url-query-encode1.png)

服务器端输出乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
ä½ å¥½
```

注：可以发现chrome浏览器对查询参数的编码也是使用utf-8

服务器端可以进行转码操作

```
System.out.println(new String(hello.getBytes("iso8859-1"),"utf-8"));
```

结论：不同浏览器对地址栏url的编码方式不同，在页面中的url的请求编码与meta中指定的编码有关系。

### 4. AJAX方式发送的GET请求对中文参数的影响

1）js代码

```
<script type="text/javascript">
    $(function(){
        $("#ajaxrequest").click(function(){//点击事件
            var url="http://localhost:8080/java_web/UrlEncodeServlet?hello=你好";
            $.ajax({type:"GET", url:url});
        });
    })
</script>
```

2）使用firefox访问

![](/images/other/encode/firefox-url-ajaxget-query-encode.png)

服务器端代码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
ä½ å¥½
```

注：可以看出firefox使用ajax发送的url对中文参数使用的是utf-8编码。

jsp页面header标签中有无指定编码都没有影响

```
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
```

3）IE浏览器访问

服务器端输出

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
C4E3BAC3
ÄãºÃ
```

注：看出IE使用ajax发送的url对中文参数使用的是GBK编码

结论：ajax的get方式在不同浏览器中对参数的编码方式不同，并且meta中指定编码也不会对其编码产生影响。

解决方法：

解决思路：

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

注：与encodeURI()的区别是，它用于对URL的组成部分进行个别编码，而不用于对整个URL进行编码。因此，"; / ? : @ & = + $ , #"，这些在encodeURI()中不被编码的符号，在encodeURIComponent()中统统会被编码。因此，如果需要对特殊的字符进行编码就采用encodeURIComponent来实现。

注2：对应的解码函数是decodeURIComponent()

后端代码：

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
System.out.println(new String(hello.getBytes("iso8859-1"),"UTF-8"));
//输出
E4BDA0E5A5BD
ä½ å¥½
你好
```

注：两种方式在后台获取的数据都是UTF-8编码的字节数组。

### 5. AJAX方式发送的POST请求对中文参数的影响

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

服务器端代码--正常输出中文

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("UTF-8")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
你好
```

注：firefox以post方式提交的数据中文使用utf-8编码，并且服务器端也能以utf-8解码，原因是在http请求头中添加了编码，服务器端会根据该编码进行解码。

```
Content-Type:"application/x-www-form-urlencoded; charset=UTF-8"
```

3）使用IE浏览器访问

![](/images/other/encode/ie-url-ajaxpost-query-encode.png)

服务器端代码--中文正常输出

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("UTF-8")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
你好
```

注：IE浏览器的post提交中文参数也是使用utf-8进行编码的，服务器能够使用utf-8解码，也是因为http请求头中添加了编码，服务器端会根据该编码进行解码。

```
Content-Type:"application/x-www-form-urlencoded; charset=UTF-8"
```

结论：ajax的post提交对中文参数使用UTF-8编码，并能在ContentType中指定编码方式，因此服务器端能够使用正确的编码方式进行解码。

注：服务器端在使用request.getParameter方法时会根据请求头的ContentType来解析编码。因此，服务器端针对post提交的数据在调用getParameter前先设置 request.setCharacterEncoding("UTF-8");在获取数据，就会以设置的编码来解码参数。

### 6. 用表单的form方式提交

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
//输出
E4BDA0E5A5BD
ä½ å¥½
```

注：可以看出浏览器端提交参数使用的是UTF-8编码，服务器端没有正确解码，原因是form提交时，header头信息中添加的contentType没有指定编码，因此服务器端不知道该用什么编码方式进行解码，因此使用iso8859-1进行了解码。

```
Content-Type:"application/x-www-form-urlencoded"
```

注：常用的解决方法是使用spring mvc编码过滤器，CharacterEncodingFilter，该过滤器主要是解决form提交表单指定中文编码为UTF-8，只能解决post提交，不能改变get的解码方式。

3）使用IE浏览器提交

服务器端代码--中文乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
ä½ å¥½
```

注：原理与firefox提交原理是相同的。

总结：form表单提交参数使用UTF-8编码，但并没有在ContentType中指定编码方式，因此服务器端仍然使用iso8859-1解码。


### 5. 如何改变服务器端对url参数的解码方式

1）通过上面的实例可以发现服务器端对查询参数的解码使用的是iso8859-1。

考虑：服务器端对uri的解码方式可以通过设置server.xml中的URIEncoding="UTF-8"，来实现，同样需要在服务器端设置某个参数来达到对查询参数的编码设置。

这个参数就是：

```
useBodyEncodingForURI="true"
```

该参数解决query String的乱码问题。useBodyEncodingForURI=true -> 使用http header中指定charset进行decode(例如：Content-Type: charset=UTF-8)，若未指定，则使用默认值ISO-8859-1。useBodyEncodingForURI=false -> 使用默认值ISO-8859-1。

2）分析useBodyEncodingForURI对服务器查询参数编码的影响（仅适用火狐浏览器测试）

服务器端代码--用来查看获取请求参数和服务器端使用的编码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
```

```
<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" useBodyEncodingForURI="true"/>
```

注：这是目前server.xml文件的设置，这也上面的例子中为什么服务器端对查询参数使用的是iso8859-1进行解码成字符串。要理解

```
String hello=request.getParameter("hello");
```

注：这行代码其实是服务器的解析字节数组获取的，即解析客户端发送的查询参数的字节数组，服务器端使用iso8859-1解码后获取的字符串。

3）不设置useBodyEncodingForURI参数（测试1）

方案一：不设置useBodyEncodingForURI，只设置URIEncoding="UTF-8"

```
<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

使用火狐访问

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=你好
```

url访问的编码

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=%E4%BD%A0%E5%A5%BD
```

服务器端代码--正确输出汉字字符

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("utf-8")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
你好
```

注：正确输出汉字，服务器端使用的是utf-8进行查询字符串解码。

方案二：修改URIEncoding="GBK"

```
<Connector URIEncoding="GBK" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

服务器端代码--汉字乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("GBK")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
浣犲ソ
```

注：可以看出服务器端对查询字符串使用的是GBK进行解码的，因为得到的字节数组与客户端访问的字节数组相同。

另外，服务器端转码获取正确的汉字字符串

```
System.out.println(new String(hello.getBytes("GBK"),"UTF-8"));
```

方案三：同时去掉URIEncoding="GBK"--汉字乱码

```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" />
```

服务器端代码--汉字乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
ä½ å¥½
```

服务器端转码

```
System.out.println(new String(hello.getBytes("iso8859-1"),"UTF-8"));
```

注：可以看出服务器端对查询字符串使用的是ISO8859-1进行解码的，因为得到的字节数组与客户端访问的字节数组相同。

总结：

useBodyEncodingForURI不设置，且URIEncoding也不设置时，使用ISO8859-1进行服务器端的查询字符串解码。

useBodyEncodingForURI不设置，而URIEncoding设置，则使用URIEncoding设置的编码访问进行服务器端的查询字符串解码。

4）设置useBodyEncodingForURI

方案一：去掉URIEncoding

```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" useBodyEncodingForURI="true"/>
```

服务器端代码--中文乱码

```
String hello=request.getParameter("hello");
        System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
        System.out.println(hello);
//输出：
E4BDA0E5A5BD
ä½ å¥½
```

服务器端转码

```
System.out.println(new String(hello.getBytes("iso8859-1"),"UTF-8"));
```

注：可以发现服务器端使用iso8859-1进行解码

方案二：设置URIEncoding为utf-8

```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" useBodyEncodingForURI="true" URIEncoding="UTF-8"/>
```

服务器端代码--中文乱码

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("iso8859-1")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
ä½ å¥½
```

服务器端转码

```
System.out.println(new String(hello.getBytes("iso8859-1"),"UTF-8"));
```

注：可以发现服务器端依然使用iso8859-1进行解码

方案三：在header头中设置Content-Type: charset=UTF-8

```
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" useBodyEncodingForURI="true" URIEncoding="UTF-8"/>
```

通过fireburg的编辑重发功能，在header头中添加Content-Type: charset=UTF-8，再次发送请求。

![](/images/other/encode/firefox-url-encode-contentType.png)

服务器端代码--中文正常输出

```
String hello=request.getParameter("hello");
System.out.println(DatatypeConverter.printHexBinary(hello.getBytes("UTF-8")));
System.out.println(hello);
//输出
E4BDA0E5A5BD
你好
```

总结：

设置useBodyEncodingForURI后，URIEncoding设置的编码类型不再对请求参数的编码有影响。请求参数使用http header中指定charset编码方式进行decode。如何header中不指定charset，则使用默认的iso8859-1进行解码。

### 5. url参数的解码方式总结

1）在tomcat的server.xml设置<Connector URIEncoding="utf-8"  />， 使用utf8对URI中出现的中文进行decode，例如http://localhost:8080/test/测试.do -> http://localhost:8080/test/%E6%B5%8B%E8%AF%95.do, 这样能够找到对应的controller方法。若不设置使用默认值ISO-8859-1

2）在tomcat的server.xml设置<Connector useBodyEncodingForURI="true"  /> 能够解决query String的乱码问题。
useBodyEncodingForURI=true -> 使用http header中指定charset进行decode(例如：Content-Type: charset=UTF-8)，若未指定，则使用默认值ISO-8859-1
useBodyEncodingForURI=false -> 使用默认值ISO-8859-1

3）若只配置了URIEncoding="utf-8"，则query string的编码也会被设置为utf-8，且http header中设置的charset不会重写这个编码。若同时设置了两项，则对于query string的编码，则与设置useBodyEncodingForURI=true的作用是一样的（会被http header中的charset重写）。

参考：http://blog.csdn.net/a285981079/article/details/49928399

### 6. 如何设置jsp响应编码

1）在jsp页面中添加

```
<%@page contentType="text/html; charset=UTF-8" %>
```

返回内容的响应头中就会出现：Content-Type:"text/html;charset=UTF-8"

2）在服务器端代码调用

```
response.setContentType("text/html; charset=UTF-8");
```

注：utf8在IE下显示乱码，必须使用UTF-8

```
<%@page contentType="text/html; charset=utf8" %>
```

### 7. URI和URL的区别

1）概念问题：

URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。

URL是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。

注：URI是以一种抽象的，高层次概念定义统一资源标识，而URL则是具体的资源标识的方式。

2）服务器端代码区别

访问地址：
```
http://localhost:8080/java_web/UrlEncodeServlet?hello=你好
```

代码执行：

```
System.out.println(request.getRequestURI());
System.out.println(request.getRequestURL());
//输出
/java_web/UrlEncodeServlet
http://localhost:8080/java_web/UrlEncodeServlet
```

注：URI实例可以代表绝对的，也可以是相对的；而URL类则不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的。

参考：http://www.ruanyifeng.com/blog/2010/02/url_encoding.html

参考：http://www.cnblogs.com/haitao-fan/p/3399018.html#3527681

参考：http://www.cnblogs.com/gaojing/archive/2012/02/04/2413626.html