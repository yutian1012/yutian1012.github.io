---
title: memcached的使用
tags: [memecached]
---

1）运维部署场景

在实际使用时，memcached内存指定多大是根据业务有多少数据要缓存来确定的。此外需要针对业务的重要性，决定是否采取负载均衡，分布式等架构。最后确定并发连接数。

注：如果采用分布式，必须要有开发人员参与，指定使用的hash算法来定位服务。

2）客户端使用方式

memcached存储的数据是以键值对的形式存储的，使用文本型的协议，比较简单。因此除了使用客户端读写数据外，甚至可以使用telnet或nc命令进行数据的读写。

### 实例

1）使用telnet做客户端

```
# 安装telnet
yum install telnet -y
# 运行telnet连接memcached服务，telnet连接时必须使用ip地址
telnet 10.0.0.8 11211
# 写入数据使用set命令，xxkey参数表示key的名称，第一个0表示过期时间，0表示不过期
# 第二个参数0表示一个标志位flag，最后参数6是存放内容的大小（字节），回车
set xxkey 0 0 6 
# 输入值不能比设置的6大，否则不能存储
hello
# 获取值
get xxkey

# 退出
quit
```

2）使用nc做为客户端向memcached服务中写入数据

```
# 安装nc命令
yum install nc -y
# 通过printf的方式向nc中输入数据，\r\n表示结尾标志
printf "set key008 0 0 10\r\nhelloworld\r\n" | nc 127.0.0.1 11211
# 读取使用get命令
printf "get key008\r\n" | nc 127.0.0.1 11211

# 删除数据使用delete命令
printf "delete key008\r\n" | nc 127.0.0.1 11211
```

注：使用set命令时设置的存储大小要与存储的value大小匹配，不能超过。