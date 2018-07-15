---
title: zookeeper为节点acl授权操作 -单机
tags: [zookeeper]
---

参考：https://blog.csdn.net/cdu09/article/details/51637451

1）权限设置方式

```
# 下面表达式中的ACL的表示方法为：scheme:id:permission，
# scheme表示身份认证，从world,auth,digest,ip中取值
# id是cheme的取值，如world下id只能为anyone，如果id书写错误提示：Acl is not valid
# permission即节点操作权限，取值为crwda
# setAcl path acl

# 设置/zk节点权限（先创建该节点）
setAcl /zk world:anyone:crwd
```

注：权限设置时需要对节点同时设置身份认证和操作权限。

2）身份认证world方式：

```
# 身份认证world方式，设置/zk操作节点为crwd
setAcl /zk world:anyone:crwd
getAcl /zk
```

注：这是节点创建后默认的权限

3）身份认证auth方式：

当验证策略为auth（本质就是digest）时，它没有id值，表示给认证通过的所有用户设置acl权限。可以通过addauth命令（addauth digest username:password）进行认证用户的添加；当使用addauth命令添加多个认证用户后，使用auth策略来设置acl，那么所有认证过的用户都被会加入到acl中。

注：只要使用auth策略设置acl权限，那么所有被认证过的用户对该节点的权限都会跟着变化，后加入的用户（addauth）不能具有之前的acl权限，需要再次使用auth来设置权限

注2：一定要在同一个session中添加addauth，auth权限才能生效

使用实例：

```
# 身份认证auth方式，并授权acl访问
# 查看当前/zk节点的权限
getAcl /zk

# 保证/zk不具备子节点
rmr /zk
create /zk "test"
getAcl /zk

# 在当前会话中没有添加认证用户直接使用auth授权，会出现错误
setAcl /zk auth::crwda
--------------
Acl is not valid : /zk
--------------

# 先添加认证用户，然后使用auth授权
addauth digest user1:user1passwd
addauth digest user2:user2passwd
setAcl /zk auth::crwda
getAcl /zk 
```

注：使用auth时，如果配置的权限是父节点会出现Authentication is not valid问题。

重新打开一个session

```
# 在上面的session中已经调用了addauth添加了认证用户，重新打开后使用auth授权，抛出异常
create /zkauth2 "test"
setAcl /zkauth2 auth::r
---------------------
Acl is not valid : /zkauth2
---------------------

# 再次调用addauth添加认证用户
addauth digest user1:user1passwd
setAcl /zkauth2 auth::r

```

常见问题：

```
# 1、session中没有使用addauth添加认证用户，直接调用setAcl授权时，抛出异常
Acl is not valid : 

# 2、当授权的节点不是叶子节点（即存在子节点时），抛出异常？？？
Authentication is not valid
```

存在的问题

addauth与setAcl？？

父目录设置auth::r后无法使用rmr删除，Authentication is not valid，并且子节点也无法删除，如何删除？？？