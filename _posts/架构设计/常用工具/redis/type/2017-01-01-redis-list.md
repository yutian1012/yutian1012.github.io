---
title: redis集合操作命令
tags: [redis]
---

1）插入操作

```
# 链表值：0--a--b--c
lpush character a
rpush character b
rpush character c
lpush character 0
# 取出1和2下标的元素
lrange character 1 2
# 取出所有元素0 -1
lrange character 0 -1
# 在指定位置处插入数据，在元素b的前面插入a
linsert character before b a
```

2）弹出值

```
# 左侧元素弹出
lpop character
# 右侧元素弹出
rpop character
```

3）删除链表元素

```
# 清空数据
flushdb
# 插入7个元素
rpush answer a b c a b d a
# 从左到右删除1个b
lrem answer 1 b
```

4）链表剪贴

```
flushdb 
rpush character a b c d e f
# 从2截取到5：c d e f
ltrim character 2 5
```

5）返回索引值

```
# 获取索引0对应的值
lindex character 0
# 链表长度
llen character
```

6）右侧弹出的同时左侧插入

该命令一般用于两个链表对接，将链表1弹出，并放置到链表2的左侧

```
rpush task a b c d
# task的d放置到job链表中
rpoplpush task job
```

7）阻塞pop

```
# brpop如果有元素则直接弹出，否则等20秒后，如果插入了元素则弹出
brpop job 20
```

注：可以开启两个客户端来进行测试。