---
title: github开源项目云收藏（thymeleaf模板）
tags: [project]
---

在相关的脚手架项目中已经引入了web和thymeleaf等相关模块的依赖，这里结合bootstrap制作前端页面显示。

利用thymeleaf布局+bootstrap前端框架搭建项目的前端。

1）引入依赖

```
<!-- thymeleaf layout布局依赖 -->
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

2）布局页面

页面一般包含五大部分

a.header（头信息，一般为图片logo等，navigation等）

b.left（左边栏目信息，一般是左侧树形结构）

c.content（中间内容显示区域）

d.sidebar（右侧工具栏区域）

f.footer（底部版权，电话等信息）

![](/images/project/favorites-web/web/layout.png)

3）创建简单布局实例

