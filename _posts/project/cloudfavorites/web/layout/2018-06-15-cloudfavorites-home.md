---
title: github开源项目云收藏（首页）
tags: [project]
---

登录云收藏系统后，会跳转到home.jsp页面，下面看一看具体的内容是如何展示的？

1）home.html页面

```
<html xmlns:th="http://www.thymeleaf.org" layout:decorate="layout">
  <head th:include="layout :: htmlhead" th:with="title='favorites'"></head>
  <body>
      <section layout:fragment="content">
      </section>
  </body>
  <script type='text/javascript'>
 </script>
</html>
```

这里使用layout:decorate指定使用哪个布局页面装饰该页面（该命令等价于data-layout-decorator）。layout:fragment指定这部分区域将要替换layout.html页面的content区域。

2）用户登录后的后台操作

用户输入用户名和密码后，点击登录按钮，会跳转到UserController类的login方法进行登录逻辑的处理，返回json对象到页面，页面js处理返回信息，并跳转到登录后的地址（具体可查看login.html页面）。

```
if(response.data.rspCode == '000000'){
     window.open(response.data.data, '_self');//成功的访问地址为：/
}
```

IndexController类的home方法会处理地址为（/）的请求，从数据库中获取基本信息并转到home.html页面上。home.html页面是使用layout.html作为布局页面进行内容渲染的。

好了，现在我们已经到home.html页面了，也能成功的看到该页面的信息。但是页面的content的区域又是如何展示的呢？

在layout.html页面的最下方定义了一段script脚本，用来获取content展示区的内容

3）content展示区内容

layout.html页面下方的script脚本有下面一句话，通过方法名，可以推断是去到standard地址下请求内容了。

```
locationUrl('/standard/my/0','home');
```

通过搜索，定位到locationUrl方法位于comm.js文件中，最终发现访问后端数据是通过XMLHttpRequest相关对象实现的。并将结果设置到content区域中

```
var xmlhttp = new getXMLObject();//js访问后台对象
function goUrl(url,params) {
    fixUrl(url,params);
    if(xmlhttp) {
        //var params = "";
        xmlhttp.open("POST",url,true);
        //设置响应回调，具体回调方法handleServerResponse查看下方定义
        xmlhttp.onreadystatechange = handleServerResponse;
        xmlhttp.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded;charset=UTF-8');
        xmlhttp.send(params);//发送请求
    }
}

//XML OBJECT
function getXMLObject() {
    var xmlHttp = false;
    try {
        xmlHttp = new ActiveXObject("Msxml2.XMLHTTP") // For Old Microsoft
                                                        // Browsers
    } catch (e) {
        try {
            xmlHttp = new ActiveXObject("Microsoft.XMLHTTP") // For Microsoft
                                                                // IE 6.0+
        } catch (e2) {
            xmlHttp = false // No Browser accepts the XMLHTTP Object then false
        }
    }
    if (!xmlHttp && typeof XMLHttpRequest != 'undefined') {
        xmlHttp = new XMLHttpRequest(); // For Mozilla, Opera Browsers
    }
    return xmlHttp; // Mandatory Statement returning the ajax object created
}
```

定义回调方法，并将信息显示到content区域

```
//回调函数
function handleServerResponse() {
    if (xmlhttp.readyState == 4) {
        //document.getElementById("mainSection").innerHTML =xmlhttp.responseText;
        var text = xmlhttp.responseText;
        if(text.indexOf("<title>Favorites error Page</title>") >= 0){
            window.location.href="/error.html";
        }else{
            //还记得layout定义区域时使用的id就是content吗？
            //<div layout:fragment="content" id="content" ></div>
            $("#content").html(xmlhttp.responseText);
        }
    }
}
```

上面一系列js方法会访问/standard/my/0。由HomeController类的standard方法实现后台逻辑。

该方法最终将collect/standard.html页面返回给浏览器，然后在调用回调显示返回的html文本。

```
//使用html方法可以在浏览器中展示相关的html文本
$("#content").html(xmlhttp.responseText);
```

4）页面js无法成功加载

修改jquery相关的引用地址

```
<script type="text/javascript" src="/webjars/jquery/2.0.0/jquery.min.js"></script>
```

具体地址信息可以查看相关jar文件的resources目录下的地址信息

![](/images/project/favorites-web/webjquery-jar-path.png)