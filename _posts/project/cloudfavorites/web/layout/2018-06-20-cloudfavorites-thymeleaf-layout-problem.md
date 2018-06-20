---
title: github开源项目云收藏（thymeleaf模板布局问题）
tags: [project]
---

1）布局文件加载问题

从别的项目中拷贝的thymeleaf布局页面，应用到系统中。访问页面无论如何都无法加载。单独访问layout布局页面可以正常查看。使用data-layout-decorate配置的布局页面，只能显示布局页面的内容，无法访问到layout布局页面使用th:replace命令引入的其他模板文件。

注：单独访问layout.html布局页面，能够获取到th:replace命令引入的其他模板文件。

原因是依赖问题，需要在依赖中引入thymeleaf-layout-dialect的依赖

解决方法：

```
<!-- thymeleaf layout布局依赖 -->
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```