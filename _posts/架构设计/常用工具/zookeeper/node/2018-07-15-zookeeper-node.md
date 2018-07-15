---
title: zookeeper文件系统（以树形结构组织）-单机
tags: [zookeeper]
---

zkClient常用的操作和文件系统大致相同，主要包括查看、新增、修改、删除、配额、权限控制等。

1）运行客户端连接zookeeper

```
zkCli.cmd  -timeout 5000  -r  -server  localhost:2181
```

2）查询操作

zkCli可以查询节点的数据和节点的状态。使用stat列出节点的状态；使用get获得节点的数据；使用ls列出节点的子节点列表；使用ls2同时列出子节点的列表和节点的状态；

注：根目录类似linux系统的根目录，使用/表示

```
# 查看根目录下的节点列表，刚创建完成后根目录下只有/zookeeper一个节点
ls /
# 查看当前节点数据并能看到更新次数等数据，可通过numChildren看到目录的节点数量
# ls2命令包含ls和stat2部分，ls展示目录下的节点列表，逗号分隔
ls2 /
# 节点状态
stat /
# 获取节点数据，get命令包含数据和stat命令2部分信息
get /
```

3）创建节点

使用create命令创建节点

```
# 创建/zk节点
create /zk "test"

# 查看根目录下的节点列表[zk,zookeeper]，即新增了/zk节点
ls /

# 在/zk下创建子节点
create /zk/node1 "node1"

# 获取/zk节点数据
get /zk

# 修改/zk节点数据
set /zk "test2"
```

注：创建相同的节点会出现提示Node already exists

create命令详解：

```
# -s表示创建顺序节点；-e表示创建临时节点 
create [-s] [-e] path data acl
```

注：zookeeper的临时节点在客户端失联后会自动被删除

4）删除节点

```
# 删除/zk节点
delete /zk

# 如果/zk节点下有多个子节点，直接使用delete不能删除，需要使用rmr，
# 其中第二r可理解为递归删除
# rmr /zk
```

