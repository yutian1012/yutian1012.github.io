---
title: 用户组管理
tags: [activiti]
---

Activiti对用户和用户组的管理提供了IdentityService组件。

### 用户组管理

1）数据对象

Group对象接口，该实例表示一条用户组数据。对应的表为ACT_ID_GROUP。该接口提供类相应字段的get和set方法。并且Group接口有一个实现类GroupEnitity。

GroupEntity提供了相应属性与数据库表结构对应

```
protected String id;//ACT_ID_GROUP的ID_列
protected int revision;//ACT_ID_GROUP的REV_列
protected String name;//ACT_ID_GROUP的NAME_列
protected String type;//ACT_ID_GROUP的TYPE_列
```

2）用户组对象的创建

使用IdentityService的newGroup方法创建对象，使用saveGroup方法保存对象到数据库中。

```
//调用newGroup方法创建Group实例
Group group = identityService.newGroup("1");
group.setName("经理组");
group.setType("manager");
//保存Group到数据库
identityService.saveGroup(group);
```

注：创建Group对象时需要传递groupID值，并且要避免与数据库中已有键值重复。

3）交由Activiti来生成主键，防止主键重复。

先使用newGroup获取对象，然后重新设置Group的ID为null，那么在调用saveGroup方法时，该方法会按照Activiti的规则为该Group生成ID。

```
//调用newGroup方法创建Group实例
Group group = identityService.newGroup("1");
group.setName("经理组");
group.setType("manager");
group.setId(null);
//保存Group到数据库
identityService.saveGroup(group);

identityService = engine.getIdentityService();
//调用newGroup方法创建Group实例
group = identityService.newGroup("1");
group.setName("经理组");
group.setType("manager");
group.setId(null);
//保存Group到数据库
identityService.saveGroup(group);
```

![](/images/book/workflow/activiti/user/usergroup.png)

注：Activiti的主键生成规则也是以递增的顺序来处理的。

4）修改用户组

修改用户组也是调用saveGroup方法，在保存前会先判断group对象的REV_值，如果该值不为0，即视为修改操作。

```
Group data=identityService.createGroupQuery().group("1").singleResult();
data.setName("经理2组");
identityService.saveGroup(data);
```

5）删除用户组

IdentityService提供了一个deleteGroup方法用于删除用户组数据。