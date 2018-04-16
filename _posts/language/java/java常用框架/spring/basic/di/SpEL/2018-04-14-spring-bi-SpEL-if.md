---
title: spring表达式结合判断条件
tags: [spring]
---

1）判断是否为空

在调用对象方法时，一般先要判断是否为null，如果对象为null调用方法会抛出异常。

为了避免出现NullPointerException，可以使用类型安全的“?.”运算符：

```
#{artistSelector.selectArtist()?.toUpperCase()}
```

这个运算符能够在访问它右边的内容之前，确保它所对应的元素不是null。所以，如果selectArtist的返回值是null的话，那么SpEL将不会调用toUpperCase方法。

注：如果selectArtist返回null，表达式的返回值将会是null。

2）使用三目运算符

```
//三目运算
#{scoreboard.score > 1000 ? "Winner!" : "Loser"}
//判断空并提供默认值，如果disc.title属性为null，则返回后面的字符串
#{disc.title ?: 'Rattle and Hum'}
```