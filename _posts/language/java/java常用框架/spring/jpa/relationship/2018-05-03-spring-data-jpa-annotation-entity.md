---
title: spring boot配置使用JPA注解
tags: [spring]
---

参考：https://docs.spring.io/spring-data/jpa/docs/2.1.0.M2/reference/html

1）表和字段信息

a.@Entity 设置在实体类上，标识一个需要被持久化的对象

b.@Id 用于声明一个实体类的属性映射为数据库的主键列

c.@GeneratedValue 用于标注主键的生成策略

注：默认情况下，JPA自动选择一个最适合底层数据库的主键生成策略

d.@Column 设置表字段信息，可设置名称，非空等限制，当然也可以不在属性上添加，该属性默认也会保存到表字段中。

2）忽略实体中的字段

在字段上使用@Transient注解，则不会与数据库表的字段对应。

```
@Entity
public class Student{
    @Transient
    private String test;//该字段不存储到数据库中
    ...
}
```

3）枚举类型存储字符串

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