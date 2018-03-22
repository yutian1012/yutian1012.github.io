---
title: redis事务与mysq事务
tags: [redis]
---

*事务提供了数据访问的一致性和可靠性。*

redis事务与mysql的事务进行类比，查看redis的事务信息。

1）mysql事务：

开启一个事务（start transaction），执行批量语句（sql），如果执行失败则回滚（rollaback），如果执行成功则提交（commit）

2）Redi事务s：

开启事务（multi），执行批量语句（普通命令），如果失败（discard取消），成功（exec）

### 实例

实例：转账功能通过事务实现，从zhao的账号下转账100到wang的账号下。

1）mysql事务实现转账

```
# 启动事务
start transaction;
# 执行转账的sql操作
update account set money=money-100 where uname='zhao';

# 如果此时出现问题，需要回滚数据
-- rollback;

# 转账到wang的账号中
update account set money=money+100 where uname='wang';

# 失败仍可以回滚
-- rollback;

# 成功提交
commit;
```

2）redis的事务处理

```
# 初始化数据
set wang 200
set zhao 700

# 启动事务，multi表示多的意思，即接下来要输入多条语句来执行
multi

# 执行转账，会将语句先放置到queue中
decryby zhao 100
incryby wang 100

# 成功则提交，会执行队列中的语句
exec
```