---
title: mysql语句过长限制
tags: [database]
---

参考：http://blog.csdn.net/superit401/article/details/77480078

1）错误提示信息

```
Got a packet bigger than 'max_allowed_packet' bytes
```

主要是在执行insert语句时，存在一个表的插入，该语句接了多个values，造成数据长度超出限制。

2）查看mysql支持的最大长度

```
show VARIABLES like '%max_allowed_packet%';

+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| max_allowed_packet       | 4194304    |
| slave_max_allowed_packet | 1073741824 |
+--------------------------+------------+
```

注：4194304即4MB的大小。

3）在连接中暂时修改限制大小

```
set global max_allowed_packet = 1024*1024*160;
```

注：限制大小改为160MB