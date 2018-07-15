---
title: redis中的Hash类型
tags: [redis]
---

### 添加hash元素

为了与redis中的key相区别，hash的key称为field（域）。

1）单个域添加

```
hset user1 name lisi
hset user1 age 28
hset user2 height 175
```

注：这里的name，age，height相当于hash的key。

2）一次添加多个hash对象的field

```
hmset user2 name wang age 10 height 100
```

### 查看元素

1）单个查看元素或元素的域值

```
# 查看hash对象
hget user1
# 查看hash对象的name值
hget user1 name
```

2）一次性获取多个值

```
# 获取user1对象的多个域值
hmget user1 name height
# 获取整个对象的所有域值
hgetall user1
```

3）获取hash对象的所有key和所有value

```
# 获取user2对象的所有域信息
hkeys user2
# 获取user2对象的所有值信息
hvals user2
```

### 删除元素或元素域

1）删除对象域

```
# 删除user1对象的age域
hdel user1 age
```

### 统计

1）查看hash元素中域的个数

```
# 统计user1对象的域个数
hlen user1
```

### 判断

1）判断Hash对象中是否存在某个域

```
# 判断user1对象是否有name域
hexists user1 name
```

### 计算

1）针对数值型的数据进行自增

```
# 将user1对象的age域加1
hincrby user1 age 1
# 增长float类型
hincrbyfloat user1 age 0.5
```