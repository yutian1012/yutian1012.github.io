---
title: spring样例
tags: [spring]
---

接触spring已经有很长一段时间了，一直没有系统的了解过spring的功能体系，只是在使用到特定模块时，才引入相关的jar文件，然后使用部分功能。

1）问题

a.没有对spring进行深入的了解，包括spring全家桶

b.spring知识掌握不扎实，只是简单的使用

c.spring源码查看，源码查看也只是抽取部分

2）目的

通过采用官方文档，以及相关书籍，包括网络资料对spring提供的各个模块，进行一个由简入深的探讨。这里通过maven来组织所有模块的样例。

3）样例组织

创建maven的父pom，通过maven的模块向父pom中不断添加spring各个模块的样例。即以一个父项目springdemo来聚合很多子项目（jpademo,jdbcdemo）等来组织我们的样例代码。

4）创建父pom，并组织创建相关的子项目

首先在eclipse中创建一个working set，方便在eclipse中管理项目的组织。

菜单命令：new--Java Working Set，命名为springdemo2018

在eclipse中创建maven项目（选择maven project），packaging选择pom。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.sjq.test</groupId>
  <artifactId>springdemo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <modules>
    <module>jpademo</module>
  </modules>
  
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
</project>
```

在父项目下创建子项目jpademo（创建时选择maven Module）

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springdemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>jpademo</artifactId>
</project>
```

5）使用父项目解决依赖

将多个子项目公用的依赖设置到父项目的pom.xml文件中，子项目中只需要引用即可，引用时无需指定version。这种方式能够解决多个子项目依赖的统一性。

注：父项目中使用dependencyManagement标签来管理依赖。

在父项目中引入spring的核心jar

```
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <spring.version>4.0.7.RELEASE</spring.version>
  </properties>
  
  <dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
  </dependencyManagement>
```

子项目中引入父项目定义的依赖

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
</dependencies>
```

注：无需指定版本信息，版本信息会从父项目管理的依赖中获取

6）将样例数据上传到github上

只需要将父项目信息上传到github上，子项目会一并上传到github上。现在github上创建一个远程仓库，然后将本地代码设置成本地仓库，然后将本地仓库与远程仓库建立管理，最后将本地仓库代码提交到远程仓库。

```
# 定位本地代码路径，执行init命令，将该目录初始化为本地仓库
git init

# 与远程仓库建立关系
git remote add origin https://github.com/yutian1012/springdemo.git
# 将远程的readme文件同步到本地
git pull origin master

# 创建.gitignore文件
# 提交本地代码到远程
git add .
git commit -m u ....
# 将本地代码同步到远程
git push origin master
```