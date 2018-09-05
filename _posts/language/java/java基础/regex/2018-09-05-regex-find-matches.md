---
title: java正则find和matches方法
tags: [java]
---

参考：http://www.runoob.com/java/java-regular-expressions.html

matches是整个字符串匹配，find结合group实现分组捕获。

注：matches方法返回true或false，判断是否整体匹配

1）find方法

find用于从一个字符串中找到匹配的方法

```
// 按指定模式在字符串查找
String line = "This order was placed for QT3000! OK?";
String pattern = "(\\D*)(\\d+)(.*)";

// 创建 Pattern 对象
Pattern r = Pattern.compile(pattern);

// 现在创建 matcher 对象
Matcher m = r.matcher(line);
if (m.find( )) {
 System.out.println("Found value: " + m.group(0) );
 System.out.println("Found value: " + m.group(1) );
 System.out.println("Found value: " + m.group(2) );
 System.out.println("Found value: " + m.group(3) ); 
} else {
 System.out.println("NO MATCH");
}
```