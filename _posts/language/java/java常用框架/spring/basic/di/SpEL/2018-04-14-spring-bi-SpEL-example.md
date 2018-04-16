---
title: spring表达式调用类的静态方法
tags: [spring]
---

1）调用静态方法

T()表达式会将java.lang.System视为Java中对应的类型，因此可以调用其static修饰的currentTimeMillis()方法。

注：T()可以访问类作用域的方法和常量。

a.访问静态方法（类方法）

```
#{T(System).currentTimeMillis()}
```

结果是计算表达式的那一刻当前时间的毫秒数。

b.访问常量

```
#{T(java.lang.Math).PI}
```

2）引用bean及bean的属性和方法

SpEL表达式也可以引用其他的bean或其他bean的属性和调用对象方法。

a.引用bean的属性值

表达式会计算得到ID为sgtPeppers的bean的artist属性：

```
#{sgtPeppers.artist}
```

b.引用bean对象，通过ID引用其他bean

```
#{sgtPeppers}
```

c.调用bean方法

```
#{artistSelector.selectArtist()}
```

3）引用系统属性

通过systemProperties对象引用系统属性：

```
#{systemProperties['disc.title']}
```

4）字面量值的引用

可以使用SpEL来表示字面量，实际可以用来表示整型、浮点数、String值以及Boolean值。

a.数值的表示

数值的表示可以使用浮点型和科学计数型方式表示。

```
//浮点数的表示
#{3.14159}
//科学计数法表示
#{9.78E4}
```

b.字符串常量表示

使用单引号或双引号来表示字符串常量

```
#{'helloworld'}
```

c.Boolean类型的数据表示

```
#{true}
```

注：SpEL表达式可以使用简单的字面量组合表示复杂的情况