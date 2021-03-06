---
title: redis的实用案例--统计用户登录情况
tags: [redis]
---

案例：1亿个用户，用户有频繁登陆的，也有不经常登陆的。如何来记录用户的登陆信息，如何来查询活跃用户（如1周登陆3次）。

思路1：

存储在登录表中（uid logtime xxx），每个用户一天记录1条，那么1条就有1亿条记录，1个星期就有7亿条记录，数据量太大，不适合存储在关系型数据库中。造成统计上出现数据量大的问题。

思路2：使用位操作

使用bit作为登陆和未登录的标志，0表示未登录，1表示登录。每天按日期生成一个位图，登陆后，把user_id位上的bit值置为1。

```
# 周一设置初始值，1亿位初始值都为0
setbit mon 100000000 0
# 周一时，user_id为3的用户登录，则第3位置1
setbit mon 3 1 
....
# 周二设置初始值
setbit tues 1000000000 0
# 周3二时，user_id为2的用户登录了，则第2位置1
set thes 2 1
...
# 一周每天都登录的用户，将一周的7天数据进行与操作，并存放到res变量中
bitop and res mon tues ...
```