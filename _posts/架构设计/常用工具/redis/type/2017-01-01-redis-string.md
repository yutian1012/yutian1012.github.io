---
title: redis字符串操作命令
tags: [redis]
---

字符串类型是编程语言和应用程序中最常见和最有用的数据类型，也是Redis的基本数据类型之一。事实上，Redis中所有的键都必须是字符串。

注：redis的key和string类型value限制均为512MB。

注2：字符串的偏移是从0开始

在线帮助文档：https://redis.io/commands#string

字符串的常用命令：set,get,append,setrange,getrange,mset,mget,setnx,getset

1）set命令的附件选项

```
set key value [ex 秒数] / [px 毫秒数] [nx] / [xx]
```

注：ex，px设置有效期。nx表示key不存在时执行，xx表示key存在时才能执行

2）set设置字符串变量

```
set word hello
set "word" "hello"
# 获取数据
get word
```

注：设置时可添加双引号，也可以不添加

3）setrange覆盖字符串的一部分

```
# 替换word对应value，并指定从哪个位置开始替换，如果后面还有值，则不会删除，而是拼接上，如果空间不够自动扩展
set word "today is sunday"
setrange word 9 rainy
```

4）append命令追加字符串

```
append word "bbb"
```

5）getrange获取字符串部分值

```
set area chinese
getrange area 1 4 
```

注：下标左侧从0开始，右侧从-1开始

6）mset一次性设置多个key

```
mset key1 v1 key2 v2
# 一次获取多个key
mget key1 key2
```

注：mset的命令是原子性的设置多个变量

7）setNx

如果某个键已经存在，那么SET命令会覆盖该键此前对应的值。有时，我们不希望在键存在的时候盲目地覆盖键；这时，我们可以使用EXISTS命令来测试键的存在性。

```
# 存在则返回1，否则返回0
exists word
```

事实上，Redis提供了SETNX命令（简称为不存在时SET），用于原子性地、仅在键不存在时设置键的值。如果键的值设置成功，则SETNX返回1；如果键已经存在，则返回0且不覆盖原来的值

```
# 为已存在的key设置内容则返回0，并且不会覆盖其值
setnx word www
```

8）getset命令，获取旧值，并重新设置一个新值

```
getset area merica
```

