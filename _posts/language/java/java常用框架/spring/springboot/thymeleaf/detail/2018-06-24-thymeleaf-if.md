---
title: thymeleaf的if判断表达式
tags: [spring]
---

1）判断对象非空

```
//这里直接调用了对象方法，并使用null来判断是否为空。
<input type="button" 
    th:if="${pageable.previousOrFirst()!=null}" value="prev" onclick="prev()">
```

注：双引号中直接使用表达式，并能够调用对象的方法。