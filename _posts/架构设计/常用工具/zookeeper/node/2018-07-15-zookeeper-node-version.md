---
title: zookeeper节点版本
tags: [zookeeper]
---

1）版本信息

所谓版本号就是对应的值的修改次数

```
版本类型        说明
dataversion     当前数据节点的数据内容的版本号
cversion        当前数据节点的子节点的版本号，c表示children
aclversion        当前数据节点的ACL变更的版本号
```

2）查看dataversion版本变化

```
# 创建数据节点
create /zk/datanode1 "datenodetest"

# 查看数据内容dataversion信息，dataversion值为0
stat /zk/datanode1

# 修改数据节点的数据
set /zk/datanode1 "datanodetest2"

# 查看数据内容dataversion信息，dataversion值为1
stat /zk/datanode1
```

3）查看cversion版本变化

```
# 创建数据节点
create /zk/datanode2 "datenodetest"

# 查看该节点的cversion，值为0
stat /zk/datanode2

# 创建子节点
create /zk/datanode2/node1 "node1"

# 查看该节点的cversion，值为1，与numChildren值相同
stat /zk/datanode1
```

4）查看节点aclversion版本变化

```
# 创建数据节点
create /zk/datanode3 "datenodetest"

# 查看该节点的aclversion，值为0
stat /zk/datanode3

# 设置节点的权限，此时aclverion值为1
setAcl /zk/datanode3 world:anyone:crwda

# 命令查看节点权限设置
getAcl /zk/datanode3
-------------------
'world,'anyone
: cdrwa
-------------------
```