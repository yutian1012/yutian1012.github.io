---
title: spring中profile的处理
tags: [spring]
---

Environment对象提供了检查哪些profile被激活的方法，可以方便的判断激活的profile，从而创建相关的对象。

1）常用的方法

```
//返回激活profile名称的数组
String[]getActiveProfiles()
//返回默认profile名称的数组
String[]getDefaultProfiles()
//如果environment支持给定profile的话，就返回true。
booleanacceptsProfiles(String...profiles)
```