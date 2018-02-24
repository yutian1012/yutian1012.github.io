---
title: 浏览器URL编码
tags: [coding]
---

这里主要涉及URL中的中文编码，以及URL查询参数中的中文编码问题。

*URI中的中文编码问题不能通过request.setCharacterEncoding设置编码实现，该方法只能实现post方式提交的数据的如何解码*

URL编码问题

1）可以在URL中正确编码的字符

```
# 字母和数字
0-9a-zA-Z
# 特殊符号
$-_.+!*'(),
```

注：双引号，某些保留字，汉字，就必须编码后使用，才可以用于URL。

### 搭建web环境

1）创建maven的web项目

```
<dependencies>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
</dependencies>
```

2）创建UrlEncodeServlet类

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
        //未解码前
        System.out.println(request.getRequestURI());
        //解码后的url信息
        System.out.println(request.getPathInfo());
    }
}
```

3）配置web.xml

```
<servlet>
    <servlet-name>UrlEncodeServlet</servlet-name>
    <servlet-class>org.sjq.test.UrlEncodeServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>UrlEncodeServlet</servlet-name>
    <url-pattern>/UrlEncodeServlet/*</url-pattern>
  </servlet-mapping>
```

### 汉字出现在URL路径中

1）访问路径（在浏览器中直接回车）：

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

3）输出内容

```
/java_web/UrlEncodeServlet/%E4%BD%A0%E5%A5%BD
/你好
```

java代码中你好对应的UTF-8编码值

```
public static void main(String[] args) throws UnsupportedEncodingException {
    System.out.println(DatatypeConverter.printHexBinary("你好".getBytes("UTF-8")));
}
//输出E4BDA0E5A5BD
```

注：可以看到utf-8编码的"你好"与uri编码的值是相同的，从而也可以确定浏览器所使用的uri编码方式是utf-8。

4）服务器端设置URI编码

*如果服务器端出现的URI地址中汉字显示乱码，只能通过服务器设置uri的编码方式解决*

在tomcat服务器中，可以设置server.xml中的URIEncoding编码。这样服务器端在获取浏览器发送的uri时，会认为浏览器使用的是utf-8对uri进行编码，因此使用相同的utf-8对其进行解码，从而在uri中的中文字符就不会出现乱码。

```
URIEncoding="UTF-8"
```

注：保证两端的编码/解码方式相同

### 查询字符串包含汉字

1）访问路径：

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=你好
```

2）火狐浏览器查看

```
http://localhost:8080/java_web/UrlEncodeServlet?hello=%E4%BD%A0%E5%A5%BD
```

![](/images/other/encode/firefox-url-query-encode1.png)

3）服务器端获取传递参数

```
protected void doService(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String hello=request.getParameter("hello");
    System.out.println(hello);
}
```

注：字符串hello输出乱码

解决方案1：修改tomcat的server.xml的Connector标签中设置属性

```
URIEncoding="UTF-8"
```

解决方案2：使用ISO8859-1编码在使用UTF-8解码

```
System.out.println(new String(hello.getBytes("iso8859-1"),"utf-8"));
```