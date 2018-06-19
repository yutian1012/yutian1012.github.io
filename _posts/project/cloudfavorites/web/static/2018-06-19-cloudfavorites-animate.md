---
title: github开源项目云收藏（动画样式）
tags: [project]
---

参考：https://daneden.github.io/animate.css/

animate.css能够在页面上实现动态效果

1）实例

```
<html>
<head>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css">
</head>
<h1 class="animated infinite bounce">Example</h1>
</html>
```

打开页面，h1标题是跳动的效果，也可以通过jquery的addClass来实现样式绑定

```
$('#yourElement').addClass('animated bounceOutLeft');
```