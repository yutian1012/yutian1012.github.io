---
title: zookeeper的安装
tags: [zookeeper]
---

ZooKeeper是用Java编写的，运行在Java环境上，因此，在部署zk的机器上需要安装Java运行环境。解压后即可使用，类似tomcat。

注：zookeeper要求Java运行环境，并且需要jdk版本1.6以上。

参考：https://www.cnblogs.com/jxwch/p/6433310.html

1）安装过程

```
cd /usr/local/src
# 下载
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz
tar -xzvf zookeeper-3.4.8.tar.gz
cd zookeeper-3.4.8
```