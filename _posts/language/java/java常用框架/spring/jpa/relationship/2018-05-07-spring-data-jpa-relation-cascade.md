---
title: spring boot jpa 对象关联性
tags: [spring]
---

表与表之间存在一对一，一对多，多对多的关系，同样映射到对象间，也存在这样的三种关系。对应的注解为@OneToOne，@OneToMany，@ManyToMany

### 注解中的常用属性

1）fetch属性，关联数据的加载策略

```
# 数据延迟加载，默认策略
fetch=FetchType.LAZY

# 非延迟加载关联数据
fetch=FetchType.EAGER
```

2）cascade属性，级联操作

一般采用CascadeType.MERGE，级联合并（级联更新）即可，默认值是均不进行关联。

```
cascade={CascadeType.PERSIST，CascadeType.MERGE,CascadeType.REFRESH,CascadeType.REMOVE}

# 级联新增，又称级联保存
CascadeType.PERSIST
# 级联更新
CascadeType.MERGE
# 级联删除
CascadeType.REMOVE
# 级联刷新
CascadeType.REFRESH
# 以上四种都是
CascadeType.ALL
```

### @JoinColumn注解

参考：https://blog.csdn.net/u010588262/article/details/76667283

@JoinColumn用于指定关联字段名称

### 主控方

所谓主控方就是能改变关联关系的一方，如用户表，与用户扩展表（该表提供了几个扩展字段，用于存储用户的大文本数据等扩展信息）。那么用户表就是主控方，用户扩展表是被控方。

注：用户的扩展信息并没有使用子表进行扩展存储。