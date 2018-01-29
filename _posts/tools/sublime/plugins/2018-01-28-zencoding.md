---
title: Emmet(zencoding)插件
tags: [sublime]
---

参考：https://docs.emmet.io/

1）安装该插件Emmet

ctrl+shift+p），输入install选中Install Package并回车，此时进入到安装package的交互界面中，输入并选中Emmet插件，然后回车进行安装。

2）快速创建html

```
# 文档中输入，然后按tab键，就能生成嵌套的html
# 外层div，class为row；内存div，class为col-md-12，连续生成5行
div.row>div.col-md-12*5

# 生成的代码
<div class="row">
    <div class="col-md-12"></div>
    <div class="col-md-12"></div>
    <div class="col-md-12"></div>
    <div class="col-md-12"></div>
    <div class="col-md-12"></div>
</div>
```