---
title: linux开机启动过程
tags: [linux]
---

1）通过rc.local配置开机启动

以配置apache启动为例

```
vim /etc/rc.d/rc.local
# 添加启动命令
/usr/local/apache/bin/apachectl start
```