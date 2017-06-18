---
title: maven创建web项目
tags: [maven,tools]
---

该项目的创建IDE工具使用的是Myeclipse，与eclipse环境略有差别。

### 1. 创建maven项目，配置相关环境

![](/images/tools/maven/mvnproject.png)

maven的web项目的目录结构（dos命令查看：tree /F），如果目录不存在，可以手动创建

```
├─main
│  ├─java
│  ├─resources
│  └─webapp
│      │  index.jsp
│      │
│      └─WEB-INF
│              web.xml
│
└─test
    ├─java
    └─resources
```

注：java目录用来放置源代码，resources目录用来放置配置文件、国际化资源等文件，webapp用来放置静态资源js，css，img以及jsp页面，ftl模板等文件。

### 2. 配置项目preferences

1）配置编码为UTF-8

```
右键项目--properties--Resource--text file encoding
```

2）配置jdk环境(本地系统的jdk)

```
右键项目--properties--java build path--libraries
```

![](/images/tools/maven/mavenprojectbuildpath.png)

### 3. 配置web部署路径

项目右击--properties--MyEclipse--Deployment Assemble设置web项目的输出部署路径。

![](/images/tools/maven/maven-deployment.png)

### 4. 设置编译器版本

```
<build>
    <finalName>webmodel</finalName>
    <pluginManagement>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
            </configuration>
        </plugin>
    </plugins>
   </pluginManagement>
</build>
```

### 5. 部署运行

测试地址：http://localhost:8080/webmodel/

### 6. 切换web.xml的版本

生成的web使用的2.3

```
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
```

修改为3.0版本

```
<plugin> 
    <artifactId>maven-war-plugin</artifactId> 
    <configuration>
        <version>3.0</version> 
    </configuration> 
</plugin>
```

查看并配置Project Facets：项目右击--properties--MyEclipse--Project Facets

![](/images/tools/maven/maven-facets.png)

生成新的web.xml文档

删除web.xml文件，项目右击--MyEclipse--Generate Deployement Descriptor Stub,会自动生成web.xml文件。

![](/images/tools/maven/maven-generatewebxml.png)

### Archive for required library

错误信息：

```
Archive for required library: 'G:/repository/joda-time/joda-time/1.6.2/joda-time-1.6.2.jar' in project 'zsy Maven Webapp' cannot be read or is not a valid ZIP file
```