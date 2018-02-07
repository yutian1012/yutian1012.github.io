---
title: hibernate中Criteria查询
tags: [hibernate]
---

Criteria是一种比hql更面向对象的查询方式。Criteria 可使用 Criterion 和 Projection 设置查询条件。
参考：http://www.cnblogs.com/mabaishui/archive/2009/10/16/1584510.html
Criteria 和 DetachedCriteria 的主要区别在于创建的形式不一样， Criteria 是在线的，是由
Hibernate Session 进行创建的；而 DetachedCriteria 是离线的，创建时无需Session。
Criteria 和 DetachedCriteria 均可使用 Criterion 和 Projection 设置查询条件。可以设置 FetchMode( 联合查询抓取的模式 ) ，设置排序方式。对于 Criteria 还可以设置 FlushModel 
（冲刷 Session 的方式）和 LockMode （数据库锁模式）。

1、通过Criteria实现查询

```
/**
* 按属性查找唯一对象
*/
@SuppressWarnings("unchecked")
protected T findUniqueByProperty(String property, Object value) {
    Assert.hasText(property);
    Assert.notNull(value);
    return (T) createCriteria(Restrictions.eq(property, value))
        .uniqueResult();
}
```

注：最终使用Criteria的uniqueResult()查询，createCriteria()是定义在父类HibernateBaseDao中的方法。

```
/**
 * 根据Criterion条件创建Criteria,后续可进行更多处理,辅助函数.
*/
protected Criteria createCriteria(Criterion... criterions) {
    Criteria criteria = getSession().createCriteria(getEntityClass());
    for (Criterion c : criterions) {//封装条件
        criteria.add(c);
    }
    return criteria;
}
```

注：使用getSession()方法获取（org.hibernate.Session对象），即数据库的操作session，createCriteria来创建Criteria，实现查询。

2、通过hql语句查询
```
public int countByEmail(String email) {
    String hql = "select count(*) from UnifiedUser bean where bean.email=:email";
    Query query = getSession().createQuery(hql);
    query.setParameter("email", email);
    return ((Number) query.iterate().next()).intValue();
}
```

1）编写hql查询语句
2）构建Query对象，通过Session的createQuery方法构成出查询对象
3）在查询对象上设置参数setParameter
4）执行查询操作

3、使用session的get方法获取对象

```
public UnifiedUser findById(Integer id) {
    UnifiedUser entity = get(id);
    return entity;
}
```

调用父类HibernateBaseDao中的get方法
```
protected T get(ID id, boolean lock) {
    T entity;
    if (lock) {
        entity = (T) getSession().get(getEntityClass(), id,LockMode.UPGRADE);
    } else {
        entity = (T) getSession().get(getEntityClass(), id);
    }
    return entity;
}
```
注：Session的get方法需要获取模型对象的Class，使用主键进行查询。

4、使用session的save方法
```
public UnifiedUser save(UnifiedUser bean) {
    getSession().save(bean);
    return bean;
}

```

5、使用session的delete方法
```
public UnifiedUser deleteById(Integer id) {
    UnifiedUser entity = super.get(id);
    if (entity != null) {
        getSession().delete(entity);
    }
    return entity;
}
```
