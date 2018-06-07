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

实例1：展示商品信息，每行一个商品记录

```
<table>
    <tr>
      <th>xh</th>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
    </tr>
    <tr th:each="prod,index : ${prods}">
      <td th:text="${index.count}"></td>
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```

实例2：下拉列表查看，下拉列表选项中显示集合字符串成员

```
<select name="sourceTable">
  <option th:each="tableName:${sourceTableList}" 
    th:text="${tableName}" th:value="${tableName}">
  </option>
</select>
```

注：${index.count}表示循环的序号

5）在js中使用

参考：https://blog.csdn.net/mygzs/article/details/52667897

```
# 在script标签上使用th:inline="javascript"属性
# Thymeleaf解析时，会解析/*[[…]]*/的内容，并把获得的值替换掉/*[[…]]*/后面的内容
/*<![CDATA[*/
  var valueType = /*[[${valueType}]]*/ 'defaultvalue';
/*]]>*/
```

6）判断条件

判断对象非空

```
<input type="button" th:if="${pageable.previousOrFirst()!=null}" value="prev" onclick="prev()">
<input type="button" th:if="${pageable.next()!=null}" value="next" onclick="next()">
```

注：这里直接调用了对象方法，并使用null来判断是否为空。

7）集合操作

参考：https://blog.csdn.net/u012485114/article/details/53906250

```
# 遍历map集合数据
# valueType是一个map集合，通过value和key属性来展示数据
<select>
    <option th:each="entry:${valueType}" th:text="${entry.value}" th:value="${entry.key}"></option>
  </select>
```