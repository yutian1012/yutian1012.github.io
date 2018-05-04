---
title: extjs入门样例
tags: [theme]
---

1）创建html页面

```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>test webjars</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/extjs/6.0.0/classic/theme-classic/resources/theme-classic-all.css" rel="stylesheet" />
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/extjs/6.0.0/ext-all.js"></script>
    <script type="text/javascript">
         Ext.onReady(function() {
             Ext.create('Ext.Panel', {
                renderTo: 'helloWorldPanel',
                height: 200,
                width: 600,
                title: 'Hello world',
                html: 'First Ext JS Hello World Program'
                });
         });
    </script>
</head>
<body>
    <div id="helloWorldPanel" />
</body>
</html>
```

2）代码分析

a.Ext.onReady方法

该方法在页面加载后执行函数，类似jquery的ready方法

b.Ext.create方法

该方法用于在Ext JS中创建对象，这里创建一个简单的面板类Ext.Panel的对象。

c.Ext.Panel

Ext JS中用于创建面板的预定义类。

3）预定义类属性

每个Ext JS类都有不同的属性来执行一些基本的功能。如Ext.Panel类的各种属性:

```
renderTo属性，此面板必须呈现的元素，即绑定到页面的什么元素上，这里绑定到了div上。 

Height宽度属性，用于提供面板的自定义尺寸。

Title属性，为面板提供标题。

Html属性，在面板中显示的html内容。
```