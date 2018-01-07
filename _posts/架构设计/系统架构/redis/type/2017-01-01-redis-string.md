---
title: redis字符串操作命令
tags: [redis]
---

1）set命令的附件选项

```
set key value [ex 秒数] / [px 毫秒数] [nx] / [xx]
```

注：ex，px设置有效期。nx表示key不存在时执行，xx表示key存在时才能执行

2）一次性设置多个key

```
mset key1 v1 key2 v2
# 一次获取多个key
mget key1 key2
```

3）setrange替换字符串

```
set word hello
# 将两个ll替换成??
setrange word 2 ??
# range超出单词长度，则填补\x00
set word hello
# 输出hello\x00\x00!
set range 7 !
```

4）append追加

```
append word @
```

5）getrange获取字符串部分值

```
set area chinese
getrange area 1 4 
```

注：下标左侧从0开始，右侧从-1开始

6）getset命令，获取旧值，并重新设置一个新值

```
getset area merica
```

7）bit位操作

```
# 语法
setbit key offset value
# 实例
# A的ascii码值为：A 65 0100 0001
# a的ascii码值为：a 97 0110 0001
set char A
# 从左侧高位开始偏移，将第2位变成1
setbit char 2 1
# 输出小写a
get char
```

注：字符串的offset最大值为2^32-1，字符串最大存储为512MB

8）位操作，位的与或非操作

```
# 设置bit位的操作规则，大写转小写
setbit lower 2 1
# 设置key为一个大写字母
set char Q
# 位的or操作，将结果设置到char变量中，操作的两个key是最后两个参数
bitop or char char lower
```
