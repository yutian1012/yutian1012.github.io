---
title: thymeleaf常用命令
tags: [spring]
---

1）引入命名空间

在页面的html元素上引入命名空间

```
<html xmlns:th="http://www.thymeleaf.org">
```

2）url链接

Thymeleaf对于URL的处理是通过语法@{…}来处理的，使用th:href标签属性。

```
# 绝对路径
<a th:href="@{http://blog.csdn.net/u012706811}">绝对路径</a>

# 相对路径，一般用于访问controller的path时使用
<a th:href="@{/}">相对路径</a>

# 一般用于访问静态资源路径
<a th:href="@{css/bootstrap.min.css}">默认访问static下的css文件夹</a>
```

链接中追加参数信息

```
# ${table.id}在链接中添加table变量的id属性值
<a th:href="@{'/tables/migrate/'+${table.id}}"></a>
```

注：当涉及到动态传递变量值时，一般是利用单引号拼接变量的形式