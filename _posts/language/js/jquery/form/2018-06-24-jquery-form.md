---
title: jquery-form异步提交表单
tags: [jquery]
---

1）jquery-form提供了异步提交表单的功能

这种方式提交表单

入门实例

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
    <script src="https://cdn.bootcss.com/jquery.form/4.2.1/jquery.form.min.js"></script>
</head>
<body>
    <form id="testform" method="post" enctype="multipart/form-data">
        <input type="text" name="username" />
        <input type="file" accept="text/xml" name="uploadFile"/>
        <input type="button" id="submitForm" value="提交"/>
    </form>
</body>
<script type="text/javascript">
    $(function(){
        $("#submitForm").click(function(){
            //8080端口服务必须是存在的，否则无法提交表单
            var url="http://localhost:8080/test";
            //控制表单按钮
            $("#submitForm").attr("disabled","disabled");
            //使用ajaxSubmit方法，会自动把表单中的文本域提交到后台
            //可以提交带文件的表单
            $("#testform").ajaxSubmit({
                type: 'post',
                async: true,
                url: url,
                success: function(response){

                }
            });
        });
    });
</script>
</html>
```