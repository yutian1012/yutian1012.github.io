---
title: maven创建web项目
tags: [maven,tools]
---

该项目的创建IDE工具使用的是Myeclipse，与eclipse环境略有差别。

### myeclipse下maven的web项目创建

1）创建maven项目，配置相关环境

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

2）配置项目preferences

配置编码为UTF-8

```
右键项目--properties--Resource--text file encoding
```

配置jdk环境(本地系统的jdk)

```
右键项目--properties--java build path--libraries
```

![](/images/tools/maven/mavenprojectbuildpath.png)

3）配置web部署路径

项目右击--properties--MyEclipse--Deployment Assemble设置web项目的输出部署路径。

![](/images/tools/maven/maven-deployment.png)

4）引入web依赖的jar包

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.0.1</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>
```

注：属性provided表示该jar包由容器提供，不需要发布到lib目录下，只在程序编译期使用。

5）设置编译器版本

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

6）部署运行

测试地址：http://localhost:8080/webmodel/

7）切换web.xml的版本

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

### eclipse下maven的web项目创建--切换web-app版本为3.0

1）web.xml中web-app头

参考：http://blog.csdn.net/z69183787/article/details/36008097

web-app.3.0版本

```
<?xml version="1.0" encoding="UTF-8"?>  
<web-app  
        version="3.0"  
        xmlns="http://java.sun.com/xml/ns/javaee"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">  
   
</web-app>
```

2）eclipse将web-app.2.3改成3.0

错误信息：Cannot change version of project facet Dynamic web

配置编译环境大于1.6

```
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.0</version>
    <configuration>
        <source>1.7</source>
        <target>1.7</target>
    </configuration>
</plugin>
```

项目右击--properties--Project Facets，如果依然不能实现Dynamic Web Module从2.3切换到3.0，只能手动进行修改。

修改项目下的.settings文件夹下的org.eclipse.wst.common.project.facet.core.xml文件

```
<installed facet="jst.web" version="3.0"/>
```

右击项目--maven--update project...