---
title: redis消息发布与订阅
tags: [redis]
---

消息的发布与订阅主要有三个重要命令：

pulish（发布频道，并在频道中发布信息）

subscribe（订阅频道）

psubscribe（正则方式监听多个频道）

在实际测试时，可启动两个或多个客户端来测试。

1）发布频道

```
# 使用pulish向频道中发表消息，news表示一个频道（channel）
pulish news 'today is sunshine'
```

2）订阅频道（在另外的客户端监听）

```
# 使用subscribe命令订阅频道，news表示一个频道（channel）
subscribe news
```

3）使用正则的方式订阅频道

```
# 使用正则匹配监听的频道，可同时监听多个匹配到的频道
psubscribe new*
```

注：p代表pattern，监听所以new开头的频道