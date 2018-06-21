---
title: 校验文件接收类型
tags: [jquery]
---

1）实例代码

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>validator test</title>
    <script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
    <script src="https://cdn.bootcss.com/jquery-validate/1.17.0/jquery.validate.min.js"></script>
    <script src="https://cdn.bootcss.com/jquery-validate/1.17.0/additional-methods.min.js"></script>
    <script src="https://cdn.bootcss.com/jquery-validate/1.17.0/localization/messages_zh.min.js"></script>
    
</head>
<body>
    <form id="fileForm" method="post" enctype="multipart/form-data">
        <input type="file" name="uploadFile" accept="image/*">
        <input type="submit" name="提交">
    </form>
</body>
<script type="text/javascript">
    $("#fileForm").validate({ //绑定表单
        rules : { //设置校验规则
            uploadFile : { //与name属性对应
                required : true
            }
        }
    });
</script>
</html>
```

2）提示文本问题：

如果上传其他类型的文件提示信息如下，是英文的提示信息。

```
Please enter a value with a valid mimetype.
```

原因是国际化文件中没有提供accept对应的中文提示，修改成中文提示

```
//在validate中配置message节点
messages:{
    uploadFile:{
        accept : "请选择图片类型的文件"   
    } 
}
```

3）查看accept方法的校验

在additional-methods.min.js文件中存在相应的校验方法。

```
a.validator.addMethod("accept",
    function(b, c, d) {
        var e, f, g, h = "string" == typeof d ? d.replace(/\s/g, "") : "image/*",
        i = this.optional(c);
        if (i) return i;

        if ("file" === a(c).attr("type") && (h = h.replace(/[\-\[\]\/\{\}\(\)\+\?\.\\\^\$\|]/g, "\\$&").replace(/,/g, "|").replace(/\/\*/g, "/.*"), c.files && c.files.length)) 

            for (g = new RegExp(".?(" + h + ")$", "i"), e = 0; e < c.files.length; e++) 
        
                if (f = c.files[e], !f.type.match(g)) return ! 1;
        
            return ! 0
    },
    a.validator.format("Please enter a value with a valid mimetype.")),
```

