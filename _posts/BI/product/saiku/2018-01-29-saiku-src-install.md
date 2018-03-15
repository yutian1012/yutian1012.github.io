---
title: saiku源码安装（未完成）
tags: [BI]
---

1）saiku的安装

下载地址：https://github.com/OSBI/saiku

将项目fork到自己的本地，然后checkout最新的一个版本，运行测试。

2）下载代码并测试运行

```
# 下载fork分支到本地
git clone https://github.com/yutian1012/saiku.git
# 查看tag
git tag -l
# 切换版本tag-3.8.8
git checkout tag-3.8.8
```

编译项目

```
mvn clean compile -Dmaven.test.skip=true

mvn clean install -Dmaven.test.skip=true
```

错误信息

```
[ERROR] Failed to execute goal on project saiku-service: Could not resolve dependencies for project org.saikuanalytics:saiku-service:jar:3.8.7: Failed
 to collect dependencies at org.saiku:saiku-query:jar:0.4-SNAPSHOT:...
```

注上面的错误信息表示不能正确获取到saiku-query相关的jar包。这部分代码并没有提供到相应的repository中。

解决方法：

下载一个安装版的saiku，从WEB/lib目录下获取改jar包，然后按照到本地的mvn仓库中。

使用mvn构建时经常会遇到这样的问题，主要原因是由于mvn仓库的地址经常发生变动，尤其是历史版本的项目，一般这种错误的解决方案就是在本地安装即可。

```
mvn install:install-file -Dfile=D:\work\installer\saiku-server\tomcat\webapps\saiku\WEB-INF\lib\saiku-query-0.4-SNAPSHOT.jar -DgroupId=org.saiku -DartifactId=saiku-query -Dversion=0.4-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=D:\work\installer\saiku-server\tomcat\webapps\saiku\WEB-INF\lib\mondrian-4.3.0.1.1-SPARK.jar -DgroupId=pentaho -DartifactId=mondrian -Dversion=4.3.0.1.1-SPARK -Dpackaging=jar

mvn install:install-file -Dfile=D:\work\installer\saiku-server\tomcat\webapps\saiku\WEB-INF\lib\iText-4.2.0.jar -DgroupId=iText -DartifactId=iText -Dversion=4.2.0 -Dpackaging=jar

mvn install:install-file -Dfile=D:\work\installer\saiku-server\tomcat\webapps\saiku\WEB-INF\lib\miredot-annotations-1.3.1.jar -DgroupId=com.qmino -DartifactId=miredot-annotations -Dversion=1.3.1 -Dpackaging=jar
```

注：安装完成后，就能在本地的repository中看到该jar包。

3）修改jar包的构件参数

```
<groupId>mondrian-data-foodmart-hsql</groupId>
<artifactId>mondrian-data-foodmart-hsql</artifactId>
<version>0.1</version>
```

修改为

```
<groupId>pentaho</groupId>
<artifactId>mondrian-data-foodmart-hsqldb</artifactId>
<version>0.2</version>
```

4）安装过程中可能会遇到多种错误

最简单的解决方案是将所有的jar包到以命令的形式安装到本地仓库，这样再使用mvn构建时，错误就只剩下插件的问题，而这样的问题一般是不经常出现的。

5）导入到eclipse中

在cmd中构建成功后，导入到eclipse工具中，方便源码的查看和修改。

![](/images/BI/product/saiku/eclipse-project.png)

导入eclipse后，我们主要关注的是saiku-webapp项目，该项目是最终要被部署到tomcat下并运行的项目。并且该项目也依赖了maven的相关模块，是查看源码的入口模块。

补充：checkout问题

```
error: Your local changes to the following files would be overwritten by checkout:
        pom.xml
        saiku-core/saiku-service/pom.xml
Please commit your changes or stash them before you switch branches.
Aborting
```

原因是checkout一个版本后，做个少量的修改，现在想切换到一个新的版本上。