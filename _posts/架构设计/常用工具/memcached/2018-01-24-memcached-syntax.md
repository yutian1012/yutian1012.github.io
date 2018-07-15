---
title: memcached语法
tags: [memecached]
---

### 语法

语法格式，第一行设置key，第二行设置value，在设置过程中需要传入4个参数。

a.key字符串，小于250个字符，不包括空格和控制符

b.flags客户端用来标识数据格式的数值，如json，xml，压缩等格式

c.exptime存活时间s，0为永远，60*60*24*30表示小于30天，单位为妙，unixtime表示大于30天。

d.bytes，指定存储数据的字节数，可以是0，即存储空串。

e.datablock，文本行，即存储的value数据。

```
<command name> <key> <flags> <exptime> <bytes>\r\n
<datablock>\r\n
```

1）set命令，无论如何都要进行存储

2）add命令，只有数据不存在时进行添加

3）replace命令，只有数据存在是进行替换

4）append命令，往后追加

5）prepend命令，在前面追加

6）cas按版本号更改