---
title: 程序安装jar到本地仓库
tags: [maven]
---

在使用mvn构建项目时，由于依赖较多，手动一个一个的处理比较麻烦，可编写一个程序完成jar包的本地安装。

1）开发过程

a.获取所有依赖

遍历项目路径下的所有pom.xml，获取所有依赖的jar文件配置信息。

```
<dependency>
    <groupId>pentaho</groupId>
    <artifactId>mondrian</artifactId>
    <version>3.6.7</version>
</dependency>
```

b.获取本地的jar文件

根据指定的文件夹获取所有的jar文件路径，与pom中配置的dependency对应。如果未找到对应的jar文件，则抛出异常，并记录相关信息。

c.获取配置的仓库集合

检索pom.xml中的仓库配置信息，同时包括默认位置处的%MAVEN_HOME%下的settings.xml中的仓库信息。

```
<repositories>
    <repository>
        <id>Analytical Labs Repo</id>
        <name>Analytical Labs Repo-releases</name>
        <url>http://repo.meteorite.bi/content/repositories/alabs-release-local/</url>
    </repository>
    ...
</repositories>
```

注：最好进行jar包的签名校验，并给出相应的提示信息

d.外网检索是否能够获取

到外网检索查看是否能够获取相应的jar包，如果能够获取jar包，则不需要安装到本地，否则记录下该插件，然后准备安装到本地。

e.安装jar到本地仓库

```
mvn install:install-file -Dfile=D:\work\installer\saiku-server\tomcat\webapps\saiku\WEB-INF\lib\saiku-query-0.4-SNAPSHOT.jar -DgroupId=org.saiku -DartifactId=saiku-query -Dversion=0.4-SNAPSHOT -Dpackaging=jar
```