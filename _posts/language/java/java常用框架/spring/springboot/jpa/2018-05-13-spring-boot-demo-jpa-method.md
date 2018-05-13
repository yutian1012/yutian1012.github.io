---
title: spring boot jpa查询方法命名
tags: [spring]
---

参考：https://blog.csdn.net/u011077231/article/details/50803795

1）方法名开头前缀

查询方法对于的单词：find、findBy、read、readBy、get、getBy。

2）查询条件字段

针对pojo类的属性名进行查询字段条件设置，如findById方法名中的Id即为对应实体的一个属性。

注：如果对象中没有对应的属性，则会抛出异常

```
Caused by: org.springframework.data.mapping.PropertyReferenceException: No property nickNamedd found for type User!
```

3）条件字段的连接

使用and、or关键字连接多个字段条件。

```
# and条件
findByUsernameAndPassword(String user, Striang pwd)；
# or条件
findByUsernameOrAddress(String user, String addr)；
```

4）条件值的设定设置

查询的条件值的设定也格式各样（大于某个值、在某个范围等等）。常用的关键字有Between、LessThan、GreaterThan、IsNull、IsNotNull、Like、NotLike、Not、In、NotIn等。

```
# Between等价于 SQL 中的 between 关键字
findBySalaryBetween(int max, int min)；

# LessThan等价于 SQL 中的 "<"
# LessThanEquals等价于Sql中的"<="
findBySalaryLessThan(int max)；

# GreaterThan等价于 SQL 中的">"
findBySalaryGreaterThan(int min)；

# IsNull等价于 SQL 中的 "is null"
findByUsernameIsNull()；

# IsNotNull等价于 SQL 中的 "is not null"
# NotNull与IsNotNull 等价；
findByUsernameIsNotNull()；

# Like等价于 SQL 中的 "like"
findByUsernameLike(String user)；

# NotLike等价于 SQL 中的 "not like"
findByUsernameNotLike(String user)；

# Not等价于 SQL 中的 "！ ="
findByUsernameNot(String user)；

# In等价于 SQL 中的 "in"
# 方法的参数可以是Collection类型，也可以是数组或者不定长参数；
findByUsernameIn(Collection<String> userList)

# NotIn等价于 SQL 中的 "not in"
# 方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
findByUsernameNotIn(Collection<String> userList)

# StartingWith等价于SQL中的like '?%'
findByNameStartingWith(String name)

# EndingWith等价于SQL中的like '%?'
findByNameEndingWith(String name);

# Containing等价于SQL中的like '%?%'
findByNameContaining(String name);

# IgnoreCase等价于调用sql的Upper函数进行比较
#   where UPPER(name)=UPPER(?)
findByNameIgnoreCase(String name)
```

5）排序

使用OrderBy关键字来指定排序的字段，并且使用Asc或Desc来指定排序的规则。OrderBy等价于 SQL 中的 "order by"

```
# 如安装salary字段升序排列
findByUsernameOrderBySalaryAsc(String user)；
```