---
title: mysql事务
tags: [database]
---

参考：http://blog.csdn.net/mangmang2012/article/details/9207007

1）执行delete语句时抛出错误

错误信息

```
Lock wait timeout exceeded; try restarting transaction
```

解决方法：关闭掉引起该事务的应用。