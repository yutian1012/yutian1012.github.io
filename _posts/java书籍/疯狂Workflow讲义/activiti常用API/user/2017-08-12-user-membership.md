---
title: 用户组和用户关系
tags: [activiti]
---

在Activiti中，一个用户可以被分配到多个用户组中，一个用户组也可以包含多个用户。因此使用中间表ACT_ID_MEMBERSHIP表来保存两者的关系数据。

另外，Activiti没有为这个表做实体的映射，需要调用IdentityService类提供的createMemberShip方法，保存用户和用户组间的关系数据。

1）设置用户与用户组的绑定

```
identityService.createMembership(user.getId(),group.getId());
```

2）解除用户与用户组的绑定关系

```
identityService.deleteMembership(user.getId(),group.getId());
```

3）查询用户组下的用户

由UserQuery类来提供，因为最终获取的是User对象

```
List<User> users=
    identityService.createUserQuery().memberOfGroup(group.getId()).list();
```

4）查询用户所属的用户组

由GroupQuery类来提供，因为最终获取的是Group对象

```
List<Group> groups=
    identityService.createGroupQuery().groupMember(user.getId()).list();
```