---
title: spring嵌套bean定义
tags: [spring]
---

在Spring中，如果某个Bean所依赖的Bean不想被Spring容器直接访问，可以使用嵌套Bean。和普通的Bean一样，使用bean元素来定义嵌套的Bean，嵌套Bean只对它的外部的Bean有效，Spring容器无法直接访问嵌套的Bean，因此定义嵌套Bean也无需指定id属性。

```
<bean id="Student" class="com.abc.Student">
    <!-- 下面是一个普通的属性 -->
    <property name="name" value="张三" />
    <!-- 下面的属性是一个嵌套的Bean，对于和Student平级的Bean来说，这个Bean是不可见的，Spring容器也无法访问 -->
    <bean class="com.abc.School" />
</bean>
```

注：嵌套Bean提高了Bean的内聚性，但是降低了程序的灵活性。