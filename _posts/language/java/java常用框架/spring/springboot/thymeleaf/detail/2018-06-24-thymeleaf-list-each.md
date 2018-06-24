---
title: thymeleaf循环变量list集合
tags: [spring]
---

1）实例1：展示商品信息，每行一个商品记录

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

注：${index.count}表示循环的序号

2）实例2：下拉列表查看，下拉列表选项中显示集合字符串成员

```
<select name="sourceTable">
  <option 
    th:each="tableName:${sourceTableList}" 
    th:text="${tableName}" 
    th:value="${tableName}">
  </option>
</select>
```

注：在option中循环显示所有option选项。