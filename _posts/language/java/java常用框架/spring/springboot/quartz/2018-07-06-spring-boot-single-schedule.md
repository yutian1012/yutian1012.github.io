---
title: spring boot 整合定时器（单机）
tags: [spring]
---

1）spring 自带的定时器

使用spring-boot作为基础框架，其理念为零配置文件，所有的配置都是基于注解和暴露bean的方式。spring自带支持定时器的任务实现。其可通过简单配置来使用到简单的定时任务。

2）配置一个configuration类，在该类中定义相应的定时任务

```
package org.ipph.spider.quartz;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
@Configurable
@EnableScheduling
public class ScheduledTasks{

    @Scheduled(fixedRate = 1000 * 30)
    public void reportCurrentTime(){
        System.out.println ("Scheduling Tasks Examples: The time is now " + dateFormat ().format (new Date ()));
    }

    //每1分钟执行一次
    @Scheduled(cron = "0 */1 *  * * * ")
    public void reportCurrentByCron(){
        System.out.println ("Scheduling Tasks Examples By Cron: The time is now " + dateFormat ().format (new Date ()));
    }

    private SimpleDateFormat dateFormat(){
        return new SimpleDateFormat ("HH:mm:ss");
    }
    
}
```

3）spring自带定时器

@Scheduled注解位于spring-context包中，由spring原生提供，支持单机定时器实现。