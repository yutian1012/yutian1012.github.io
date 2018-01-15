---
title: redis应用
tags: [redis]
---

### 实战项目：仿微博系统

1）模块划分：

用户模块：用户登录，用户注册，用户退出，个人主页

微博模块：热点微博信息，发微博，查看评论。

2）数据层次

使用redis和mysql，并实现redis与mysql相结合，完成冷热数据的交换。

### 微博项目的key设计

1）全局相关的key设计

```
# 自增型的userid，全局的用户主键信息。使用incr进制自增操作
global:userid 

# 自增型的postid，全局的微博信息主键信息。使用incr进制自增操作
global:postid  
```

注：redis中设计全局自增字段

2）用户相关的key的设计

```
# userid列，作为前缀存在，不存储具体列数值
user:userid:*
# username列
user:userid:*:username
# password列
user:userid:*:password
# authsecret列
user:userid:*:authsecret
```

3）微博数据相关的key设计

方式一：直接使用key存储多个列

```
# postid列，作为前缀存在，不存储具体列数据
post:postid:*
# userid列
post:postid:*:userid
#username列
post:postid:*:username
# time列
post:postid:*:time
# content列
post:postid:*:*content
```

注：查询微博信息时，微博的发送时间等字段也要分别查询，效率低下

方式二：使用hash结构存储post信息

```
# 存储的hash结构
# array('userid'=>$user['userid'],'time'=>time(),'content'=>$content)
post:postid:$postid # 指向hash结构数据
```

4）推送表相关的key设计

```
# 每个人都直接接收到关注对象推送的数据集合
receivepost:$userid #执行微博id
```

5）关注表相关的key设计

```
# 使用集合存储关注的用户信息
following.$userid #指向userid
```

6）粉丝表相关的key设计

```
# 使用集合存储被关注的用户信息
follower:$userid #指向userid
```

### 总结：

通过项目可以看出使用redis的难度并不大，重点是由传统的mysql设计思想转变为key-value数据库的设计思想。

redis中集合操作是要经常使用的，必须熟练掌握。