---
title: github开源项目云收藏（提示弹框控件）
tags: [project]
---

参考：https://blog.csdn.net/qq_38008713/article/details/77803861

官网：http://codeseven.github.io/toastr/

1）toastr插件提示实例

```
<html>
<head>
    <link rel="stylesheet" href="http://cdnjs.cloudflare.com/ajax/libs/toastr.js/latest/css/toastr.min.css">
    <script src="http://libs.baidu.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://codeseven.github.io/toastr/build/toastr.min.js"></script>
</head>
<script type="text/javascript">
$(function(){
    toastr.info('xxxx');
});
</script>
</html>
```

2）错误信息

HTML文档的字符编码未声明。如果该文件包含 US-ASCII 范围之外的字符，该文件将在某些浏览器配置中呈现为乱码。页面的字符编码必须在文档或传输协议层声明。

原因：

这是由于没有对HTML文档的字符编码进行声明，导致该文件包含US-ASCII范围之外的其他字符，所以在某些浏览浏览器的配置中会出现乱码，因此我们必须在文档或传输协议层对页面的字符编码进行声明，这样html中的中文在浏览器中显示的时候就不会乱码。

解决方法:在head标签中添加下面的

```
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
```