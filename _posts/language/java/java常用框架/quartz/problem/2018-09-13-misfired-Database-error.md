---
title: 定时器Error handling misfires
tags: [quartz]
---

参考：https://blog.csdn.net/gklifg/article/details/27089605?utm_source=tuicool&utm_medium=referral

1）错误信息

```
ERROR [QuartzScheduler_scheduler-dell2-PC1536801357191_MisfireHandler] LocalDataSourceJobStore.manage(3940) | MisfireHandler: Error handling misfires: Database error recovering from misfires.
org.quartz.JobPersistenceException: Database error recovering from misfires. [See nested exception: java.sql.SQLException: connection holder is null]
    at org.quartz.impl.jdbcjobstore.JobStoreSupport.doRecoverMisfires(JobStoreSupport.java:3197)
    at org.quartz.impl.jdbcjobstore.JobStoreSupport$MisfireHandler.manage(JobStoreSupport.java:3935)
    at org.quartz.impl.jdbcjobstore.JobStoreSupport$MisfireHandler.run(JobStoreSupport.java:3956)
```

注：这里使用了alibaba的数据库链接池com.alibaba.druid.pool.DruidDataSource。

2）定位错误信息

这里主要涉及到quartz任务调动框架对系统的异常及崩溃后处理机制。即recovery和misfired机制处理。

查看MisfireHandler类的执行