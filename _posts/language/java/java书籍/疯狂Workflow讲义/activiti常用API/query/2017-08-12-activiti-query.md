---
title: Activiti数据查询
tags: [activiti]
---

Activiti为对象提供了相关的Query查询对象，一般通过相关的Service获取Query对象，然后设置条件，最后调用list、singleResult等方法返回查询结果，也可以使用orderByXXX方法对象查询结果进行排序。

1）Query接口和AbstractQuery抽象类

Query是全部查询对象的父接口，该接口定义了若干个基础方法，各个查询对象均可以使用这些公共方法，包括设置排序方式、数据量统计（count），列表、分页和唯一记录查询。

Query接口有一个实现类AbstractQuery，各类数据的查询对象均继承自该抽象类。

2）Query接口提供的常用方法

asc：设置查询结果的排序方式为升序

count：计算查询结果的数据量

desc：设置查询结果的排序方式为降序

list：封装查询结果，返回相应类型集合。默认按照主键（ID_列）升序排序。

listPage：分页返回查询结果

singleResult：查询单条符合条件的数据，如果查询不到，则返回null；如果查询到多条则抛出异常。

2）排序

Query提供了asc方法和desc方法，但在调用这两个方法前，必须告诉Query对象，是按何种条件进行排序。如按照ID排序，就需要先调用相应的查询对象的OrderByXXX方法，然后在跟上asc或desc方法。

在AbstractQuery对象中存在checkQueryOk方法，用来检测

```
protected void checkQueryOk() {
    if (orderProperty != null) {
      throw new ActivitiException("Invalid query: call asc() or desc() after using orderByXX()");
    }
}
```

使用方式

```
List<Group> datas=identityService.createGroupQuery().orderByGroupId().asc().list();
```

注：调用asc排序方法前先调用orderByXXX方法

3）排序问题

在Activiti的设计中，每一个数据表的主键均被设计为字符型，这样的设计使得Activiti各个数据表的主键可以灵活设置，但如果使用数字字符串作为主键，那么按照ID排序会带来问题。

结果集：1,2,3...12,13..
排序为：1,12,13...2,3..

可以使用Mysql语句进行排序，并在Order by ID_后添加"+0"，将数字字符串转换为数值。

注：activiti的API中并没有给出相应的解决方案。

4）多字段排序

排序字段的覆盖问题

```
identityService.createGroupQuery().orderByGroupId().orderByGroupName().asc().list();
```

注：这种方式的调用会使orderByGroupId的升序无效。

正确的调用方式

```
identityService.createGroupQuery().orderByGroupId().asc().orderByGroupName().asc().list();
```

注：如果调用了orderByXXX方法却没有立即调用asc方法或desc方法，则该排序条件会被下一个设置的查询条件所覆盖。

5）不同Query实体类设置了不同的查询条件和排序条件

GroupQuery类分别设置了groupId，groupName，groupNameLike等查询条件。

GroupQuery类分别设置了orderByGroupId，orderByGroupName等排序字段。