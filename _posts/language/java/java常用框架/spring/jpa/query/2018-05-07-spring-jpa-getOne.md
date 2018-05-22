---
title: SpringDateJpa使用JpaRepository方法出现空指针异常的问题
tags: [spring]
---

直接使用getOne取得对象，加断点看以一下这个对象是个hibernate的代理对象，而不是实体，里面值都是null。

注：这里直接对应dao层类做的单元测试，并没有添加事务。

1）getOne的使用场景（单元测试类）

```
@Test
public void testGetOne() {
    //数据库中真实存在的记录id
    Long id=1L;
    TableModel table=tableDao.getOne(id);
    System.out.println(table.getFrom());
}
```

注：batchLogDao继承了JpaRepository接口，并且该类是空的，没有提供任何其他方法。

2）异常信息

```
org.hibernate.LazyInitializationException: could not initialize proxy - no Session
    at org.hibernate.proxy.AbstractLazyInitializer.initialize(AbstractLazyInitializer.java:155)
    ......
```

错误的主要原因还是在于懒加载的问题。

3）使用findById方法代替

```
@Test
public void testGetOne() {
    Long id=1L;//数据库中真实存在
    TableModel table=tableDao.findById(id).get();
    
    System.out.println(table.getFrom());
}
```

4）使用事务注解方式实现

在测试类上添加事务注解的方式也可以正确的获取数据

```
@Test
@Transactional
public void testGetOne() {
    Long id=1L;//数据库中真实存在
    TableModel table=tableDao.getOne(id);
    
    System.out.println(table.getFrom());
}
```

5）问题

有时候即使添加上Transactional依然无法解决问题，具体原因待进一步查找...