---
title: redis集合操作命令
tags: [redis]
---

列表是应用程序开发中非常有用的数据类型之一。列表能够存储一组对象，因此它也可以被用作栈或者队列。Redis中的列表更像是数据结构世界中的双向链表。

1）插入操作（lpush,rpush,linsert）

LPUSH：将元素添加到列表的左端。

实现lpush左端单值插入和多值插入：

```
# 从左端插入1个字符
lpush character a
# 从左端插入2个字符
lpush character b c

# 查看列表中的数据
lrange character 0 -1
# 输出为 c b a
```

RPUSH：将元素添加到列表的右端。

实现rpush 右端单值和多值插入：

```
flushdb

# 从右端插入1个字符
rpush character a
# 从右端插入2个字符
rpush character b c

# 查看列表中的数据
lrange character 0 -1
# 输出为 a b c
```

LINSERT：将元素插入到列表的支点/枢轴元素（pivotalelement）之前或之后。

实现linsert在指定位置处插入元素：

```
在指定位置处插入数据，在元素b的前面插入a
linsert character before b d

# 查看列表中的数据
lrange character 0 -1
# 输出为 a d b c
```

2）查看列表(lrange,lindex)

lrange获取列表指定范围的元素，起始下标为0，获取的是start和end下标元素范围内的元素，包括start和end元素。

```
flushdb 
rpush character a b c

# 获取列表的所有元素，输出a b c
lrange character 0 -1

# 获取列表的前2个元素，输出a b
lrange character 0 1
```

lindex获取集合中指定下标元素

```
# 获取列表中下标为1的元素，输出b
lindex character 1
```

3）删除链表元素（lpop,rpop,ltrim,lrem）

```
flushdb
rpush character a b c

# 左侧元素弹出，输出a
lpop character
# 右侧元素弹出，输出c
rpop character

# 查看元素，只剩一个b
lrange character 0 -1
```

lrem删除元素，指定删除元素的个数count，以及删除元素的值value

```
flushdb
rpush character a b b c

# 从左到右删除1个b
lrem character 1 b

# 查看元素，输出 a b c，删除了1个b元素
lrange character 0 -1
```

ltrim按照范围裁剪集合，保留的是指定范围的元素，其余元素被从列表中移除

```
flushdb
rpush character a a b c c

# 裁剪元素，获取起始下标为1，结束下标为3的元素，元素数量为3
ltrim character 1 3

# 查看元素，输出 a b c
lrange character 0 -1
```

4）修改集合下标的元素值（lset）

```
flushdb
rpush character a a c

# 修改下标1元素值为b
lset character 1 b

# 查看元素，输出 a b c
lrange character 0 -1
```

5）列表长度(llen)

```
# 链表长度
llen character
```

6）右侧弹出的同时左侧插入到另一个列表中(rpoplpush)

该命令一般用于两个链表对接，将链表1弹出，并放置到链表2的左侧

```
flushdb
rpush task a b c d
# 将元素d弹出，并放置到job列表中
rpoplpush task job

# 输出job列表，输出d
lrange job 0 -1
```

7）阻塞弹出（blpop,brpop）

```
# brpop如果有元素则直接弹出，否则等20秒后，如果插入了元素则弹出
brpop job 20
```

注：可以开启两个客户端来进行测试。