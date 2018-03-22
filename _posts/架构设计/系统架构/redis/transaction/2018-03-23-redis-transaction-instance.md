---
title: redis事务实例
tags: [redis]
---

redis如何实现买票事务？

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

注：以上的方式可能会出现ticket票的数量成为-1，不能很好的控制并发下的变量同步操作。

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