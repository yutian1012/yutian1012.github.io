---
title: redis的orderset有序集合操作
tags: [redis]
---

需要指定一个排序因子，否则无法实现排序。

1）添加元素，添加元素时指定score（排序因子）

```
# 添加元素前需要指定排序因子，这里的数值就是排序因子
zadd class  12 lily 13 lucy 18 lilei 6 poly
```

2）查看集合

```
# 查看有序集合
# 取出第0名到第3名中的元素
zrange class 0 3
# 按score来获取，获取score为13和18之间的元素
zrangebyscore class 13 18
# 按score获取，并指定limit，从1号元素开始取，并获取2个元素
zrangebyscore class 1 20 limit 1 2
# 查看元素以及元素的score，使用withscore选项
zrange class 1 3 withscores
```

3）查看元素排名

```
# 给定元素获取元素的排名
zrank class lucy
# 反序排列获取元素的排名
zrevrank class lucy
```

4）删除元素

```
# 根据score来删除，删除score位于13到18之间的元素
zremrangebyscore class 13 18
# 根据排名删除元素，删除排名为0到2之间的元素
zremrangebyrank class 0 2
# 指定值删除
zrem class lucy
```

5）元素统计

```
# 统计元素个数
zcard  class
# 统计score值在10到18之间的元素
zcout class 10 18
```

6）集合间的运算

合并集合，并在集合合并时对score进行运算（sum，min，max）

```
zadd lisi 3 cat 5 dog 6 horse
zadd wang 2 cat 6 doc 8 horse 1 donkey
# 获取交集，并对交集元素的score进行求和，2指定需要进行计算的集合数为2个集合
zinterstore result 2 lisi wang aggregate sum
# 可对不同集合元素设置不同权重，lisi集合的权重为2，wang集合的权重为1
zinterstore result 2 lisi wang weights 2 1 aggregate sum
```