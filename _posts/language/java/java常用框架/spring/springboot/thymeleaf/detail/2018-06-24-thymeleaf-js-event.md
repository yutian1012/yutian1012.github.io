---
title: thymeleaf绑定控件事件
tags: [spring]
---

1）设置click事件

```
<a th:onclick="'javascript:exportSql('+${table.id}+');'">导出执行sql</a>
```

注：onclick使用双引号设置表达式。表达式的值是使用单引号表示的字符串，并使用+拼接具体的对象值。