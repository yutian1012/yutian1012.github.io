---
title: 遍历json对象中的属性
tags: [js]
---

1）遍历json数据对象

```
var jsonStr='{"name":"xxx",age:22}';

var json = eval('(' + jsonStr + ')');

for (var name in json) {
    var value = json[name];
    alert(name+"--"+value);
}
```

注：在for循环中使用in关键字变量对象的属性。