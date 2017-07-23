---
title: maven目录结构
tags: [maven]
---

### maven的约定

1）在项目的根目录中放置pom.xml文件

2）在src/main/java目录中放置项目的主代码

3）在src/main/resources目录中放置项目资源文件，如spring, hibernate配置文件

4）在src/test/java目录中放置项目的测试代码

5）在src/test/resources目录中放置测试资源文件

6）在src/main/webapp目录中防止web应用文件目录

7）在target目录中存放项目输出

### 使用Archetype生成项目骨架

1）maven3中运行命令

```
mvn archetype:generate -DarchetypeCatalog=internal
```

2）选择使用的archetype

![](/images/book/maven/ch3/archetype-generate.png)

实际上在运行插件maven-archetype-plugin，运行该命令会出现很多可用的archetype供选择，包括Appfuse项目的archetype，JPA项目的Archetype等。每一个archetype前都会对应一个编号。同时命令行会提示一个默认的编号，其对应的archetype为maven-archetype-quickstart。

3）输入项目的坐标信息

选择maven-archetype-webapp生成目录结构，会要求输入项目的groupId，artifactId，version以及包名package。

![](/images/book/maven/ch3/archetype-generate2.png)

4）查看创建的项目

```
E:\WORK\TESTPROJECT-AGGREGATOR
│  pom.xml
│
└─src
    └─main
        ├─resources
        └─webapp
            │  index.jsp
            │
            └─WEB-INF
                    web.xml
```

注：发现生成的目录中没有test目录（测试代码和测试资源存放目录），如果需要可以手动创建项目的目录，在运行mvn命令时，会自动识别。