---
title: zookeeper为节点acl权限设置 -单机
tags: [zookeeper]
---

参考：https://www.cnblogs.com/fesh/p/5798353.html

Zookeeper对权限的控制是节点级别的，而且不继承，即对父节点设置权限，其子节点不继承父节点的权限。

注：zookeeper的权限是通过身份认证和操作权限共同设置的。

1）节点操作权限

ZK的节点有5种操作权限：CREATE、READ、WRITE、DELETE、ADMIN。（增、删、改、查、管理权限）

注：这5种权限简写为crwda（即：每个单词的首字符缩写）

这5种权限中，delete是指对子节点的删除权限，其它4种权限指对自身节点的操作权限

2）身份认证

身份的认证有4种方式：

```
world：默认方式，相当于全世界都能访问，world下只有一个id，即anyone
auth：代表已经认证通过的用户
digest：即用户名:密码这种方式认证，这也是业务系统中最常用的
ip：使用Ip地址认证，ZK服务器会将addr的前bits位与客户端地址的前bits位来进行匹配验证权限
```

3）权限设置

权限设置时需要对节点同时设置身份认证和操作权限。

```
# 下面表达式中的ACL的表示方法为：scheme:id:permission，
# scheme表示身份认证，从world,auth,digest,ip中取值
# id是cheme的取值，如world下id只能为anyone，如果id书写错误提示：Acl is not valid
# permission即节点操作权限，取值为crwda
# setAcl path acl

# 设置/zk节点权限（先创建该节点）
setAcl /zk world:anyone:crwd
```

注：设置节点权限后，可发现节点的aclVersion值发生变化

3）实例操作

```
# 创建节点
create /zk "test"

# 查看节点的acl设置，从输出信息中可以看到身份认证为world，节点操作权限为cdrwa
getAcl /zk
--------------
'world,'anyone
: cdrwa
---------------

```