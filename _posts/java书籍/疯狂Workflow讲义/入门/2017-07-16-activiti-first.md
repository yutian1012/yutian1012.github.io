---
title: activiti的一个简单实例
tags: [activiti]
---

1）创建配置文件activiti.cfg.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 流程引擎配置的bean -->
    <bean id="processEngineConfiguration"
        class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/act" />
        <property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="root" />
        <property name="databaseSchemaUpdate" value="drop-create" />
    </bean>
</beans>
```

注：StandaloneProcessEngineConfiguration类使用spring容器来管理和注册bean对象。默认会查找classpath路径下的activiti.cfg.xml文件。

2）创建流程文件

使用eclipse的activiti designer插件来设计流程图。将创建的流程图文件放在classpath路径下的bpmn/First.bpmn，后缀为.bpmn。

菜单：new---activiti diagram

![](/images/book/workflow/activiti/firstflow.png)

注：虽然文件的后缀为.bpmn，用文本编辑器打开可以发现文件内容是xml，该xml描述了流程的定义，流程节点，流程转向，以及标识信息。

3）创建数据库

```
create database act;
```

注：需要事先把数据库创建好，数据库表可以在spring容器启动时动态创建。只需要将activiti.cfg.xml配置文件中的databaseSchemaUpdate属性修改为drop-create即可。

4）创建测试类

```
package org.crazyit.activiti;

import org.activiti.engine.ProcessEngine;
import org.activiti.engine.ProcessEngines;
import org.activiti.engine.RepositoryService;
import org.activiti.engine.RuntimeService;
import org.activiti.engine.TaskService;
import org.activiti.engine.task.Task;

/**
 * 第一个流程运行类
 * @author yangenxiong
 */
public class First {
    public static void main(String[] args) {
        // 第一步：创建流程引擎
        ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
        // 第二步：获取activiti的个各个服务组件实例
        //得到流程存储服务组件
        RepositoryService repositoryService = engine.getRepositoryService();
        // 得到运行时服务组件
        RuntimeService runtimeService = engine.getRuntimeService();
        // 获取流程任务组件
        TaskService taskService = engine.getTaskService();
        // 第三步：部署流程文件
        repositoryService.createDeployment().addClasspathResource("bpmn/First.bpmn").deploy();
        // 第四步：启动流程，流程的id为process1
        runtimeService.startProcessInstanceByKey("process1");
        // 查询第一个任务
        Task task = taskService.createTaskQuery().singleResult();
        System.out.println("第一个任务完成前，当前任务名称：" + task.getName());
        // 完成第一个任务
        taskService.complete(task.getId());
        // 查询第二个任务
        task = taskService.createTaskQuery().singleResult();
        System.out.println("第二个任务完成前，当前任务名称：" + task.getName());
        // 完成第二个任务（流程结束）
        taskService.complete(task.getId());
        task = taskService.createTaskQuery().singleResult();
        System.out.println("流程结束后，查找任务：" + task);
    }
}
```

5）依赖jar包

activiti-engine-5.10.jar，c3p0-0.9.1.2.jar，commons-dbcp-1.4.jar，commons-logging-1.1.1.jar，commons-pool-1.5.4.jar，log4j-1.2.17.jar，mybatis-3.1.1.jar，mysql-connector-java-5.1.22-bin.jar，spring-asm-3.1.2.RELEASE.jar，spring-beans-3.1.2.RELEASE.jar，spring-core-3.1.2.RELEASE.jar

![](/images/book/workflow/activiti/firstdependencylib.png)

6）常用类

ProcessEngine类加载默认的流程引擎配置文件activiti.cfg.xml，实例化流程实例。

RepositoryService主要用于管理流程的资源（流程定义xml文件，即.bmpn设计的文件），如流程定义文件部署到数据库中，进而被流程引擎加载。

RuntimeService主要用于流程运行时的流程管理，如根据流程id启动流程。

TaskService主要用于管理流程任务，如流程任务的查询，执行等操作。

注：这里获取流程服务组件都是通过ProcessEngine实例获取的，可以与spring容器结合，直接将流程服务组件初始化到spring容器中。使用时直接从spring容器中获取。