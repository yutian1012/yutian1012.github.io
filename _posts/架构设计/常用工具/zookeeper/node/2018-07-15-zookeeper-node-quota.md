---
title: zookeeper为节点分配配额 -单机
tags: [zookeeper]
---

zookeeper的配额用于限制节点的数据内容和子节点的个数。对于超出配额，系统不会导致插入或者修改不成功，但是会在 zookeeper.out 中生成错误日志。

1） setquota为节点分配配额

```
# -n表示限制子节点的个数
# -b表示限制数据值的长度
# val表示个数
# path表示想要进行设置的那个node；
setquota -n|-b val path
```

2）实例：配置-n

```
# 创建节点
create /zk/quotanode "quota"

# 分配该节点下最多创建三个子节点
setquota -n 3 /zk/quotanode 

# 进行quotanode下节点创建
create /zk/quotanode/node1 "node1"
create /zk/quotanode/node2 "node2"
create /zk/quotanode/node3 "node3"
create /zk/quotanode/node4 "node4"
```

注：实际上配额超限了之后，zookeeper只会在日志中进行警告记录，而不会抛出异常 。

3）实例：配置-b

```
# 删除之前的配额设置
delquota /zk/quotanode 

# 分配该节点的数据值长度不大于3
setquota -b 3 /zk/quotanode

# 设置节点数据值
set /zk/quotanode "12345"
```

4）查看和删除配额设置

```
# 查看配额，会显示配额绝对路径
listquota /zk/quotanode

# 删除配额信息
delquota /zk/quotanode 
```

5）错误信息：

```
Command failed: java.lang.IllegalArgumentException: /zk/quotanode has a parent /zookeeper/quota/zk/quotanode which has a quota
```

注：只能删除quota配额，并且在一个节点上只能配置一个配额数据，不能叠加。

