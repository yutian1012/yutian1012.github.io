---
title: spring值的注入
tags: [spring]
---

1）规定字符串的数据值注入

```
@Bean
public CompactDisc sgtPeppers(){
    return new BlankDisc("Sgt.Pepper's Lonely Hearts Club Band","The Beatles");
}
```

注：这里的字符串是静态传入的，是硬编码在配置类中的

2）spring提供了2种动态传参方式

a.属性占位符（property placeholder）

b.spring表达式（spEL）

