---
title: linux查看tomcat运行情况
tags: [linux]
---

1）定位服务

```
netstat -tlnp | grep 8080

# 输出
tcp        0      0 :::8080       :::*        LISTEN      16507/java   
```

注：p表示实例id

根据pid查询服务

```
ps -aux | grep 16507
```

2）通过名称查询

```
ps -ef | grep tomcat
```