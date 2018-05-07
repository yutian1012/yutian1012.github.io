---
title: spring boot thymeleaf视图模板使用
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

3）提示

html中输入th:没有自动提示的现象。

4）循环

```
<table>
    <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
    </tr>
    <tr th:each="prod : ${prods}">
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```