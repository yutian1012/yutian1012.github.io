---
title: redis事务
tags: [redis]
---

redis的事务时比较简单的，从某种意义上讲redis不支持事务回滚。原因是，开启事务后将要执行的语句会放置到队列queue中，如果队列中语句本身没有语法错误，但因为某个操作的变量类型不匹配而失败，exec执行队列命令后，最终能够执行的语句会正常执行，而类型不匹配导致错误的语句不会执行，并且抛出错误。这个错误被redis跳过了，没有整体回滚事务，而正常执行的语句却是成功的。再次执行discard也不能回滚已经成功执行了的语句。

exec时，会执行队列中已存在的命令，并且会关闭掉队列queue，此时不允许再向queue中添加任何命令。

### 事务的概念

事务提供了数据访问的一致性和可靠性。与mysql的事务进行类比，查看redis的事务信息。

mysql：开启（start transaction）,语句（sql），失败（rollaback），成功（commit）

Redis：开启（multi），语句（普通命令），失败（discard取消），成功（exec）

### 事务实例

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

### 事务回滚的3种情况

multi后面的语句中，语句出错有2种情况：语法存在问题；语法本身没有问题，但适用对象有问题；手动执行discard命令进行回滚。

1）语法问题的回滚操作

这种情况下，执行exec时会报错，所有语句得不到执行，即回滚了队列中已经成功执行的语句。

```
flushdb

# 初始化数据
set wang 200
set zhao 700

# 启动事务
multi

# 执行转账，会将语句先放置到queue中
decryby zhao 100

# 执行一个不存在的命令，导致语法错误
asdf

# 执行提交，这时会报错，并且事务进行了回滚
exec
```

2）针对类型不匹配操作的错误

这种方式语句的语法本身没有问题，但操作类型不匹配，exec会执行正确的语句，并跳过不适当的语句，这样就无法保证事务的原子性。

```
flushdb

# 初始化数据
set wang 200
set zhao 700

# 启动事务
multi

# 执行转账，会将语句先放置到queue中
decryby zhao 100

# 执行sadd向集合中添加元素a，但wang是一个字符串类型的数据，并不是集合，因此报错
sadd wang a

# 执行提交，这时会报错，但事务不会回滚
exec
```

3）执行discard命令进行回滚

```
flushdb

# 初始化数据
set wang 200
set zhao 700

# 启动事务
multi

# 执行转账，会将语句先放置到queue中
decryby zhao 100

# 回滚，取消队列中的命令
discard
```

### redis如何实现买票事务

现在我正在买票（ticket-1，money-1），而票只剩1张了。如果在我multi后和exec之前，票被别人买了（ticket=0），该如何观察这种情况并不在提交。

1）有问题的代码

```
flushdb

# 初始化数据
set ticket 1
set lisi 300
set wang 300

# lisi开始买票
multi

# 买票，票的数量减1
decr ticket
# lisi账号上钱也要减100
decrby lisi 100

# 在提交前，开启另一个窗口连接，模拟wang买票
exec
```

开启另一个窗口，模拟其他人买票

```
#开启事务
multi

# wang买票，票的数量减1
decr ticket
decrby wang 100

exec
```

2）悲观的想法

现在可能有很多人在抢票，在我付款前需要将ticket锁上，不允许其他人再来操作。

3）乐观的想法

没有那么多的人在抢票，在需要付款前只需要注意有没有人更改了ticket的值就可以了。同watch命令监视同步的变量值，从而实现事务的一致性。

```
flushdb

# 初始化数据
set ticket 1
set lisi 300
set wang 300

# 先监视ticket变量
watch ticket

# lisi开始买票
multi

# 买票，票的数量减1
decr ticket
# lisi账号上钱也要减100
decrby lisi 100

# 提交，观察的值ticket如果在队列执行前发生了改变，则会回滚而不执行
exec
```

注：使用unwatch命令取消监视

### 监视命令

watch命令可以监视对象，与事务命令配合实现同步操作。unwatch命令能够取消监视的对象。