---
title: redis的通信（RESP）
tags: [redis]
---

REdis Serialization Protocol（ RESP，Redis序列化协议）。

1）linux安装nc

```
yum install nc
```

实例一：利用nc发送PING命令

```
# 首先是要客户端连接，发送ping命令查看输出结果
./bin/redis-cli
ping

# 使用nc按照RESP协议格式发送ping命令
echo -e "*1\r\n\$4\r\nPING\r\n" | nc 127.0.0.1 6379
```

注：可以看到nc访问输出和直接使用客户的输出内容是一样的。

实例二，设置一个整数并将其加一

```
# 利用客户端访问
./bin/redis-cli
set a 1
incr a
get a
del a

# 利用nc使用RESP协议发送命令
echo -e "*3\r\n\$3\r\nset\r\n\$1\r\na\r\n\$1\r\n1\r\n" | nc 127.0.0.1 6379
echo -e "*2\r\n\$4\r\nincr\r\n\$1\r\na\r\n" | nc 127.0.0.1 6379
```

2）命令格式解析

向Redis服务器发送了*1\r\n\$4\r\nPING\r\n，命令开头的星号表示这是一个数组类型。

```
*1中的1代表这个数组的大小

\r\n（CRLF）是RESP中每个部分的终结符

\$4之前的反斜杠是$符号的转义字符，$4表示接下来是四个字符组成的字符串。

PING为字符串本身
```

注：字符串值与\r\n之间是没有反斜杆的。