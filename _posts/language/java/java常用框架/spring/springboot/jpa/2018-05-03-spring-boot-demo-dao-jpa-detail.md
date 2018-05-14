---
title: spring boot配置使用JPA注解
tags: [spring]
---

参考：https://docs.spring.io/spring-data/jpa/docs/2.1.0.M2/reference/html

1）忽略实体中的字段

在字段上使用@Transient注解，则不会与数据库表的字段对应。

```
@Entity
public class Student{
    @Transient
    private String test;//该字段不存储到数据库中
    ...
}
```

2）枚举类型存储字符串

默认枚举类型的字段会存储枚举的int值，如果想存储枚举对应的字符串可以使用注解@Enumerated

```
@Entity
public class BatchModel implements Serializable{
    @Column
    @Enumerated(EnumType.STRING)  
    private BatchStatusEnum status;
    ...
}
```