---
title: thymeleaf循环变量map集合
tags: [spring]
---

1）map集合操作

参考：https://blog.csdn.net/u012485114/article/details/53906250

```
# 遍历map集合数据
# valueType是一个map集合，通过value和key属性来展示数据
<select>
    <option 
        th:each="entry:${valueType}" 
        th:text="${entry.value}" 
        th:value="${entry.key}">
    </option>
</select>
```
