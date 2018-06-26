---
title: js访问vue中定义的变量
tags: [vue]
---

1）通过vue对象访问变量

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <script src="https://cdn.bootcss.com/vue/2.5.16/vue.min.js"></script>
</head>
<body>
    <div id="app">
        <!-- 表单控件绑定vue变量 -->
        <input type="text" name="hobby" v-model="hobby">
    </div>
    
</body>

<script type="text/javascript">
    var app = new Vue({
        el: "#app",
        data: {
            hobby:'football'
        }
    });
    console.log(app.hobby);//js中可以直接使用vue对象访问内部定义的变量
</script>
</html>
```