---
title: quartz定时器在web容器中使用
tags: [web-util]
---

### 1. 创建listener监听容器

```
public class SchedulerContextListener implements ServletContextListener {

    public void contextDestroyed(ServletContextEvent arg0) {
        // shutdown Quartz scheduler
        Scheduler scheduler = null;
        try {
            scheduler = StdSchedulerFactory.getDefaultScheduler();
        } catch (SchedulerException e) {
        }
        if (scheduler != null) {
            try {
                scheduler.shutdown();
            } catch (SchedulerException e) {
            }
        }
    }

    /* (non-Javadoc)
     * @see javax.servlet.ServletContextListener#contextInitialized(javax.servlet.ServletContextEvent)
     */
    public void contextInitialized(ServletContextEvent arg0) {
        // does nothing
    }

}
```

注：容器关闭时关闭Quartz调度服务。

### 2. 启动quartz

web.xml中配置

```
<servlet>
    <servlet-name>QuartzInitializer</servlet-name>
    <servlet-class>org.quartz.ee.servlet.QuartzInitializerServlet</servlet-class>     
    <init-param>
        <param-name>shutdown-on-unload</param-name>
        <param-value>true</param-value>
    </init-param>  
    <init-param>
        <param-name>start-scheduler-on-load</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>      
</servlet>
```