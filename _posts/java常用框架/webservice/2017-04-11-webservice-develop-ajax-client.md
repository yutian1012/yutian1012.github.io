---
title: webservice服务使用ajax方式实现客户端调用
tags: [webservice]
---

### 1. 在jsp页面中编写ajax请求

```
<%@page contentType="text/html; charset=UTF-8" %>
<html>
<body>
<h2>Hello World!</h2>
<input type="text" id="msg" />
<input type="button" onclick="sendAjaxWS();" value="通过ajax调用webservice服务"/>
</body>
<script type="text/javascript">
var xhr;
function sendAjaxWS(){
    xhr = new ActiveXObject("Microsoft.XMLHTTP");
   //指定ws的请求地址
    var wsUrl = "http://10.33.6.199:8989/WS_Server/Webservice";
    //手动构造请求体
    var requestBody='<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://webservice.web.ipph.org/">'+
                        '<soapenv:Header/>'+
                        '<soapenv:Body>'+
                            '<web:sayHello>'+
                                '<arg0>'+document.getElementById("msg").value+'</arg0>'+
                            '</web:sayHello>'+
                        '</soapenv:Body>'+
                    '</soapenv:Envelope>';
    //打开连接
    xhr.open("POST",wsUrl,true);
    //重新设置请求头xhr.setRequestHeader("content-type","text/xml;charset=utf8");
    //设置回调函数
    xhr.onreadystatechange = _back;
    //发送请求
    xhr.send(requestBody);
}

//定义回调函数
function _back(){
    if(xhr.readyState == 4){
         if(xhr.status == 200){
              var ret = xhr.responseXML;
              //解析xml
              var eles = ret.getElementsByTagName("return")[0];
              alert(eles.text);
         }
    }
}
</script>
</html>
```

注：只能在IE浏览器中执行，因为ActiveXObject对象是IE浏览器中才支持的，而在火狐中不能支持。

兼容其他浏览器，IE7已经包含了XMLHttpRequest，隐藏能兼容的只有IE6浏览器

```
function getXhr(){ 
    var xhr = null; 
    if(window.XMLHttpRequest){ 
        //非ie浏览器6
        xhr = new XMLHttpRequest(); 
    }else{ 
        //ie浏览器6
        xhr = new ActiveXObject('Microsoft.XMLHttp'); 
    } 
    return xhr; 
}
```

IE浏览器报错，错误信息：XMLHttpRequest: 网络错误 0x80070005, 拒绝访问 js错误

解决方法：Internet选项--安全--本地Intranet，将该区域的允许级别设置为低。

修改回调方法，针对

```
//定义回调函数XMLHttpRequest返回值类型的处理
function _back(){
    if(xhr.readyState == 4){
         if(xhr.status == 200){
             var ret = xhr.responseXML;
              //解析xml
              var eles = ret.getElementsByTagName("return")[0];
              alert(eles.childNodes[0].nodeValue);
         }
    }
}
```

注：针对火狐浏览器仍然无法访问，Cannot handle HTTP method: OPTIONS

![](/images/java_structure/webservice/webserviceajaxoptions.png)

### jquery的post方式提交

```
<%@page contentType="text/html; charset=UTF-8" %>
<html>
<head>
    <script type="text/javascript" src="js/jquery.min.js"></script>
</head>
<body>
<h2>Hello World!</h2>
<input type="text" id="msg" />
<input type="button" onclick="sendAjaxWS();" value="通过ajax调用webservice服务"/>
</body>
<script type="text/javascript">
function sendAjaxWS(){
    var wsUrl = "http://10.33.6.199:8989/WS_Server/Webservice";
    var requestBody='<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://webservice.web.ipph.org/">'+
                        '<soapenv:Header/>'+
                        '<soapenv:Body>'+
                            '<web:sayHello>'+
                                '<arg0>'+document.getElementById("msg").value+'</arg0>'+
                            '</web:sayHello>'+
                        '</soapenv:Body>'+
                    '</soapenv:Envelope>';
    $.ajax({
        url:wsUrl,
        type:"POST",
        dataType:"xml",
        data:requestBody,
        complete:function(data){
            alert(data);
        },
        contentType: "text/xml; charset=\"utf-8\""
    });
}
</script>
</html>
```

注：IE下能正常运行

问题：火狐下不能正常运行，Cannot handle HTTP method: OPTIONS

原因：火狐在提交时自动变成了options方式提交，而webservice不支持这种方式访问。

### 3. 使用ajax的建议

尽量不要使用ajax去调用webservice，IE涉及到跨域问题（需要调低安全级别），火狐浏览器在调用webservice时使用OPTIONS方式，而不是POST方式，服务器端无法正确处理OPTIONS请求。