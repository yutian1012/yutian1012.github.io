---
title: mybatis一级缓存
tags: [mybatis]
---

mybatis分为2级缓存

一级缓存的作用域scope是SqlSession

二级缓存，也称作全局缓存，作用域global scope的缓存

查询数据的话，先从二级缓存中拿数据，如果没有的话，去一级缓存中拿，一级缓存也没有的话再查询数据库。