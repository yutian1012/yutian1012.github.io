---
title: redis通用操作命令
tags: [redis]
---

### 常用操作

1）字符串操作

```
# 设置字符串
set site xxx.ee.scd
# 获取值
get site
```

2）查看系统的key

```
# 查看当前有多少个key，keys命令后接一个pattern
# 常用通配符：* ? []
keys *
keys sit[e|y]
# 随机获取一个key
randomkey
# 判断某个key是否存在
exists site
```

3）查看类型

```
set age 20
# 使用type命令查看类型
type age
```

4）删除

```
# 空格分隔多个key
del age site
# 重命名key
rename site site2
# 获取key
get site2
# 重命名，防止覆盖已有的key，将site改名为search，并且search不存在时才能修改成功
renamenx site search
# 删除库中的所有值
flushdb
```

### redis中数据库的概念

在配置文件redis.conf中存在database的配置项，默认为16，即启动了16个数据库。

1）默认使用的数据库是0号数据库

```
set site xxxx
# 查看当前数据库下的key，能够查看到site
keys *
# 切换数据库，切换到1号数据库
select 1
# 再次查看数据库中的key，不能找到site
keys *
```

2）将0号数据库的key移动到1号数据库

```
# 将site移动到1号数据库中
move site 1
```

### 有效时间

redis默认是没有有效期的，默认是作为数据库来使用的。

```
# 查看key的有效期，-1表示永久有效，单位是秒
ttl site
# 设置有效期，设置site存活500秒
expire site 500
# 设置毫秒存活时间
pexpire site 90000
# 查看存活的毫秒时间
pttl site 
# 设置永久有效，使用persist持久命令
persist site
```