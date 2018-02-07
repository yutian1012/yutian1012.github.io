---
title: 环境搭建和运行
tags: [activiti]
---

### 源码导入方法

创建maven项目方便查看流程的具体执行过程。

1）设置pom文件依赖

```
<dependencies>  
    <dependency>  
      <groupId>junit</groupId>  
      <artifactId>junit</artifactId>  
      <version>4.12</version>  
      <scope>test</scope>  
    </dependency>  
    <dependency>
        <groupId>org.activiti</groupId>
        <artifactId>activiti-engine</artifactId>
        <version>5.10</version>
    </dependency>
    <dependency>  
        <groupId>org.activiti</groupId>  
        <artifactId>activiti-bpmn-converter</artifactId>  
        <version>5.11</version>  
    </dependency>  
    <dependency>  
        <groupId>org.activiti</groupId>  
        <artifactId>activiti-bpmn-model</artifactId>  
        <version>5.11</version>  
    </dependency>  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <version>5.1.34</version>  
    </dependency>  
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>

<repositories>
    <repository>
        <id>activiti</id>
        <url>https://artifacts.alfresco.com/nexus/content/repositories/public/</url>
    </repository>
</repositories> 
```

2）拷贝源码文件到maven项目中

如查看第九章的startProcess项目，可在maven测试项目中添加包ch09，然后将startProcess项目代码文件拷贝到ch09包下，资源文件信息拷贝到maven项目的resources目录下。测试运行

![](/images/book/workflow/activiti/maven-code.png)

3）配置数据源信息

在activiti.cfg.xml中配置数据信息，并设置数据库更新。

```
<property name="databaseSchemaUpdate" value="true"/>
```

注：这种方式能够自动创建数据库表，实际运行实例时需要把值改为drop-create。

4）下载activiti相关的源码包

```
G:\Workspaces-workflow\workflow
```

### 安装bpmn文件的设计插件

参考：http://blog.csdn.net/qq_33547950/article/details/54926435

菜单：help--install from site

![](/images/book/workflow/activiti/eclipse-activiti-plugin.png)

安装完成后重启