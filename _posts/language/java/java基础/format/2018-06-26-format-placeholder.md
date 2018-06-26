---
title: format格式化占位符
tags: [java]
---

参考：https://blog.csdn.net/thc1987/article/details/17528093

format方法可以对字符串进行指定样式输出。

1）查看下面的方法

```
public static void formatDate() {
    String s2 = String.format("%1$tY-%1$tm-%1$te", new Date());  
    System.out.println(s2);  
}
```

注：正确输出当前日期，样式为：yyyy-MM-dd