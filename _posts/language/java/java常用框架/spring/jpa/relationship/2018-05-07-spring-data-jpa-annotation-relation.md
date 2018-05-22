---
title: spring data jpa注解
tags: [spring]
---

javax.persistence提供的注解类

1）关联关系设定

a.@OneToOne(cascade=CascadeType.ALL,optional=true)

一对一的关系，可设置表间的级联操作，如级联保存，级联删除等，optional表示是否允许外键为空。

b.@OneToMany(cascade=CascadeType.ALL)

一对多的关系，如用户与订单的关系（一个用户包含多个订单），一的一段可以设置级联操作。该注解一般用在集合属性上，集合对象为多的一段。

注：在一对多里面,无论是单向还是双向,映射关系的维护端都是在多的那一方

c.@ManyToOne

多的一的关系，用来关联出一的一段的数据。没有级联操作。

d.@ManyToMany

多对多的关系，如订单与商品的关系（一个订单中包含多个商品，同一类型的商品可以包含在不同的订单中），可设置级联操作。与@JointTable关联使用指定中间表名，以及相关的外键字段名称。

2）关联键设定

a.@JoinColumn用于设定表关联键，与关联关系注解结合使用。

一对一（@OneToOne）和多对一（@ManyToOne）的@JoinColumn注解的都是在"主控方"，即本表存放外键字段，指向关联表的主键。

一对多（@OneToMany）的@JoinColumn注解在"被控方"，即一的一方，指的是关联表中指向本表的外键名称，外键存放在关联表中。

b.@JoinTable

使用该注解可以用来设置关联表名，以及关联中该实体对于的字段名(joinColumns)，甚至可以直接指定对方字段名(inverseJoinColumns)。