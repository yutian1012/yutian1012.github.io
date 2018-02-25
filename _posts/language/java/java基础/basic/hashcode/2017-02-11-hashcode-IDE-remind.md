---
title: Hashcode和equals在IDE中设置提醒
tags: [basic]
---

为了保障equals和hashcode的一致性问题，可以在IDE中设置提醒，即equals和hashcode，不允许只实现其中一个，而不实现另一个。

1）eclipse中默认忽略该提醒

配置Eclipse来检测实现了equals方法但是没有实现hashCode方法的类，并显示错误。不幸的是，此选项默认是指为“忽略”。

2）eclipse设置方法

```
Preferences --> Java -->Compiler --> Errors/Warnings
```

菜单定位到设置页面，然后用快速筛选器来搜索“hashcode”。将Ignore值设置为warning或errors就会给出相应的提醒。
