---
title: mysql中timestamp
tags: [database]
---

目前使用的是mysql5.7版本

1）问题，创建数据库表时报错

```
1067 - Invalid default value for 'STARTDATE'
```

2）建表语句

```
CREATE TABLE `bpm_agent_setting` (
  `ID` bigint(20) NOT NULL COMMENT '主键',
  `AUTHID` bigint(20) DEFAULT NULL COMMENT '授权人ID',
  `AUTHNAME` varchar(100) DEFAULT NULL COMMENT '授权人',
  `CREATETIME` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '授权时间',
  `STARTDATE` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '开始时间');
```

3）原因：

timestamp类型取值范围：1970-01-01 00:00:00 到 2037-12-31 23:59:59

注：在mysql5.6以及之前的版本中是可以接收0000-00-00 00:00:00。

4）修改语句

```
CREATE TABLE `bpm_agent_setting` (
  `ID` bigint(20) NOT NULL COMMENT '主键',
  `AUTHID` bigint(20) DEFAULT NULL COMMENT '授权人ID',
  `AUTHNAME` varchar(100) DEFAULT NULL COMMENT '授权人',
  `CREATETIME` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '授权时间',
  `STARTDATE` timestamp NOT NULL DEFAULT '1970-01-01 00:00:00' COMMENT '开始时间');
```