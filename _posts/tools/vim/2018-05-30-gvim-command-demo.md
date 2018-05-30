---
title: gvim替换命令实例
tags: [vim]
---

参考：https://blog.csdn.net/lanchunhui/article/details/51588198

1）将逗号替换成换行符

```
:% s/,/\r/g
```

2）前后追加特定的字符

```
# 开头位置增加delete from 
:% s/^/delete from /g
# 末尾位置增加分号
:% s/$/;/g
```

3）删除空白行

```
:g/^$/d
```