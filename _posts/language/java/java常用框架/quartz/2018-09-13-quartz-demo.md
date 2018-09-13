---
title: Quartz定时器的使用案例
tags: [quartz]
---

1）创建maven项目

```
<groupId>org.sjq</groupId>
<artifactId>quartzDemo</artifactId>
<version>0.0.1-SNAPSHOT</version>

<build>  
 <plugins>  
     <!--JDK版本 -->  
     <plugin>  
         <groupId>org.apache.maven.plugins</groupId>  
         <artifactId>maven-compiler-plugin</artifactId>  
         <configuration>  
             <source>1.8</source>  
             <target>1.8</target>  
             <encoding>UTF-8</encoding>  
             <showWarnings>true</showWarnings>  
         </configuration>  
     </plugin>  
 </plugins>  
</build>  
```

2）引入依赖

```
<dependencies>
      <dependency>
         <groupId>org.quartz-scheduler</groupId>
         <artifactId>quartz</artifactId>
         <version>2.2.1</version>
      </dependency>
      <dependency>
          <groupId>org.quartz-scheduler</groupId>
          <artifactId>quartz-jobs</artifactId>
          <version>2.2.1</version>
      </dependency>   
  </dependencies>
```

3）实例代码

创建Job类

```
package ort.sjq.scheduler;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class MyJob implements org.quartz.Job {
    public MyJob() {}

    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.err.println("Hello World!  MyJob is executing.");
    }
}
```

创建调度器，并调度执行工作

```
package ort.sjq.scheduler;

import org.quartz.JobDetail;
import org.quartz.Scheduler;
import org.quartz.SchedulerException;
import org.quartz.Trigger;
import org.quartz.impl.StdSchedulerFactory;
import static org.quartz.JobBuilder.*;
import static org.quartz.TriggerBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
/**
 * Scheduler通过工厂类StdSchedulerFactory创建
 * Scheduler（调度程序，日程安排程序）
 * 需要向调度程序中指定触发器（Trigger，包括触发时机等）和调度任务（JobDetail）
 * 
 * 构造Job，使用JobBuilder提供的静态方法构造链来设置相应的调度工作对象
 * 构造Trigger，使用TriggerBuilder提供的静态方法构造链来设置相应的触发器对象
 */
public class SchedulerDemo {
    public static void main(String[] args){
        
          try {
            // Grab the Scheduler instance from the Factory
            Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
            // and start it off
            scheduler.start();
            
            // define the job and tie it to our MyJob class
            JobDetail job = newJob(MyJob.class).withIdentity("job1", "group1").build();
            
            // Trigger the job to run now, and then repeat every 40 seconds
            Trigger trigger = newTrigger().withIdentity("trigger1", "group1")
                  .startNow().withSchedule(simpleSchedule().withIntervalInSeconds(40)
                  .repeatForever()).build();
        
            // Tell quartz to schedule the job using our trigger
            scheduler.scheduleJob(job, trigger);
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
    }
}
```