---
title: hsqldb数据库文件后缀
tags: [hsqldb]
---

In H2 version 1.3.x, the database file databaseName.h2.db is the default. (The storage engine "PageStore" is used).

In H2 version 1.4.x, the database file databaseName.mv.dbis the default. (The storage engine "MVStore" is used). The MVStore is still beta right now (November 2014). But you can disable the MVStore by appending ;mv_store=false to the database URL.
