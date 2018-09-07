---
title: mybatis MappedStatement类
tags: [mybatis]
---

MappedStatement类在Mybatis框架中用于表示XML文件中一个sql语句节点，即一个select、update或者insert标签。

Mybatis框架在初始化阶段会对XML配置文件进行读取，将其中的sql语句节点对象化为一个个MappedStatement对象。