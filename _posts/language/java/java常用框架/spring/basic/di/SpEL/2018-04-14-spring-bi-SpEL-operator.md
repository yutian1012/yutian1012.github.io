---
title: spring表达式的运算符
tags: [spring]
---

1）算术运算符

```
算术运算:+、-、*、/、%、^
```

实例：计算了circlebean中所定义圆的周长

在这里PI的值乘以2，然后再乘以radius属性的值，这个属性来源于ID为circle的bean。

```
#{2*T(java.lang.Math).PI*circle.radius}
```

注：该例子展现了如何将简单的表达式组合为更为复杂的表达式。

2）String类型的+运算符

当使用String类型的值时，“+”运算符执行的是连接操作，与在Java代码中是一样的。

```
#{disc.title + ' by ' + disc.artist}
```

3）比较运算符

```
比较运算:<、>、==、<=、>=、lt、gt、eq、le、ge

逻辑运算:and、or、not、│
```

SpEL提供了比较运算符，用来在表达式中对值进行对比。比较运算符有两种形式，符号形式和文本形式。

实例：要比较两个数字是不是相等

```
//符号形式
#{counter.total == 100}
//文本形式
#{counter.total eq 100}
```

注：在xml中一般使用文本形式的比较运算符，避免与xml中的符号冲突

4）三目运算符(ternary)

```
条件运算: ?:
```

实例：

```
//三目运算
#{scoreboard.score > 1000 ? "Winner!" : "Loser"}
//判断空并提供默认值，如果disc.title属性为null，则返回后面的字符串
#{disc.title ?: 'Rattle and Hum'}
```

5）正则表达式

SpEL通过matches运算符支持表达式中的模式匹配。matches运算符对String类型的文本（作为左边参数）应用正则表达式（作为右边参数）。

```
#{admin.email matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.com'}
```
