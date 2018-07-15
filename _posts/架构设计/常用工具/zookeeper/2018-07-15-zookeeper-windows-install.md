---
title: zookeeper的安装（windows）-单机
tags: [zookeeper]
---

1）下载

官网：http://zookeeper.apache.org/

注：下载tar.gz后缀的压缩包，并没有提供zip压缩文件。

注2：zookeeper是运行在jdk环境下的，运行前需要安装jdk并配置相应的环境变量。

2）解压

将zookeeper解压到E:\work\installer\zookeeper-3.4.12目录下

重命名conf下的zoo_sample.cfg文件为zoo.cfg，并修改配置文件zoo.cfg

```
# 修改dataDir，在安装目录下创建data目录
dataDir=E:/work/installer/zookeeper-3.4.12/data
# 配置日志输出路径
dataLogDir=E:/work/installer/zookeeper-3.4.12/log
```

3）启动zookeeper服务

双击安装目录下/bin/zkServer.cmd文件启动服务

4）使用客户端工具连接

安装目录下/bin/zkCli.cmd

```
# 运行
zkCli.cmd  -timeout 5000  -r  -server  localhost:2181

# 退出节点
quit
```

5）常用命令

```
# zkCli.cmd连接服务后，按h查看帮助信息
h
```