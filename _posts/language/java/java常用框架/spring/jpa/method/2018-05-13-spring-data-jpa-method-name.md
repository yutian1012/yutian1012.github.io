---
title: spring boot jpa查询方法命名
tags: [spring]
---

参考：https://blog.csdn.net/u011077231/article/details/50803795

Spring Data会根据Repository方法的签名来实现持久化类的细节。

Repository接口方法是由一个动词、一个可选的主题（Subject，因为有参数化类型设置，因此主题是可选的）、关键词By以及一个断言所组成（可多个关键字，关键字之间使用And/OR连接符）。

```
public interface SpitterRepository extends JpaRepository<Spitter, Long> {
    Spitter findByUsername(String username);   
    ...
    //多个关键字
    List<Spitter> findByUsernameOrFullNameLike(String username, String fullName);
}
```

注：在findByUsername()这个样例中，动词是find，断言是Username，主题并没有指定，暗含的主题是Spitter。

1）方法名开头前缀

查询方法对应的动词：find、read、get以及count。

其中，动词get、read和find是同义的，这三个动词对应的Repository方法都会查询数据并返回对象。而动词count则会返回匹配对象的数量，而不是对象本身。

2）查询条件字段（对应实体属性名）

针对pojo类的属性名进行查询字段条件设置，如findById方法名中的Id即为对应实体的一个属性。

注：如果对象中没有对应的属性，则会抛出异常

```
Caused by: org.springframework.data.mapping.PropertyReferenceException: No property nickNamedd found for type User!
```

注：参数的名称是无关紧要的，但是它们的顺序必须要与方法名称中的操作符相匹配。

3）条件字段的连接

使用and、or关键字连接多个字段条件。

```
# and条件
findByUsernameAndPassword(String user, Striang pwd)；
# or条件
findByUsernameOrAddress(String user, String addr)；
```

4）条件值的设定设置

查询条件必须引用一个属性，同时还可以指定一种比较操作。如果省略比较操作符的话，那么这暗指是一种相等比较操作。要对比的属性值由相应顺序中方法的参数值提供。

查询的条件值的设定也格式各样（大于某个值、在某个范围等等）。常用的关键字有Between、LessThan、GreaterThan、IsNull、IsNotNull、Like、NotLike、Not、In、NotIn等。

```
IsAfter、 After、 IsGreaterThan、 GreaterThan 
IsGreaterThanEqual、 GreaterThanEqual 
IsBefore、 Before、 IsLessThan、 LessThan 
IsLessThanEqual、 LessThanEqual 
IsBetween、 Between 
IsNull、 Null 
IsNotNull、 NotNull 
IsIn、 In 
IsNotIn、 NotIn 
IsStartingWith、 StartingWith、 StartsWith 
IsEndingWith、 EndingWith、 EndsWith 
IsContaining、 Containing、 Contains 
IsLike、 Like 
IsNotLike、 NotLike 
IsTrue、 True 
IsFalse、 False 
Is、 Equals 
IsNot、 Not
```

注：针对string类型的数据有时也会添加忽略字符大小写的限制，使用关键字ignoringCase/ignoresCase

实例：

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

# true条件
int countProductsByDiscontinuedTrue();
```

5）排序

使用OrderBy关键字来指定排序的字段，并且使用Asc或Desc来指定排序的规则。OrderBy等价于 SQL 中的 "order by"

```
# 如按照salary字段升序排列
findByUsernameOrderBySalaryAsc(String user)；
# 可设置多个排序条件
List<Spitter> readByFirstnameOrLastnameOrderByLastnameAscFirstnameDesc(String firstname,String lastname);
```

6）去重操作

方法名在查询主题时使用Distinct关键字，则在实现时会将查询记录进行去重操作。

如果查询的主题对象本身就以Distinct开头呢？这是一种特殊的情况，如果要查询数据可不加主题，那么便不会去重。

注：在命名持久化对象时要注意，不要与spring data的查询领域关键字冲突。