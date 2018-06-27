---
title: bootstrap样式
tags: [bootstrap]
---

1）宽度前缀

超小屏幕 手机 (<768px)；小屏幕 平板 (≥768px)；
中等屏幕 桌面显示器 (≥992px) ；大屏幕 大桌面显示器 (≥1200px)

```

容器      None （自动）   750px       970px       1170px
类前缀     .col-xs-    .col-sm-    .col-md-    .col-lg-
```

注：列数最大为12，

注2：可在同一个字段上设置多个值，如.col-xs-12 .col-md-8（这样同时兼容多个屏幕）。

2）设置表格的宽度

```
<table class="table table-hover table-bordered">
    <thead>
        <tr>
          <th class="col-md-1">#</th>
          <th class="col-md-3">目标表字段</th>
          <th class="col-md-1">值来源</th>
          <th class="col-md-3">数据值</th>
          <th class="col-md-1">日志字段</th>
          <th class="col-md-2">备注</th>
          <th class="col-md-1">操作</th>
        </tr>
    </thead>
    ...
</table>
```

注：通过col-md来设置宽度，列数为12个

3）问题

现在希望，如果列数大于12列，那么超出的部分能否隐藏，并显示扩展按钮，点击则展开

