---
title: pentaho源码搭建
tags: [BI]
---

参考：http://blog.csdn.net/jxpaiwp/article/details/51498515

选择使用7.1版本，在构建项目时需要使用jdk1.8，系统中有很多语法都是依赖1.8版本。

在parent模块的pom.xml文件中添加编译信息

```
<build>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.2</version>
        <configuration>
            <source>${java.version}</source>
            <target>${java.version}</target>
            <!-- 显示警告信息 -->
            <showWarnings>true</showWarnings>
        </configuration>
    </plugin>
</build>
```

### 下载并构建

1）下载源码

源码下载地址：https://github.com/pentaho/pentaho-platform

使用7.1源码版本运行（切换分支）

```
git clone https://github.com/pentaho/pentaho-platform.git
# 查看当前分支信息
git branch
# 查看项目远程可用分支信息
git branch -a
# 切换分支到7.1版本
git checkout 7.1
```

切换标签tag

```
# 先git clone整个仓库，然后git checkout tag_name就可以取得tag对应的代码了
# 查看所有的标签
git tag
# 切换代码到相应的tag上
git checkout 7.1.0.8-R
```

注：最好checkout标签，否则在branch上操作可能需要修改很多的文件，才能正常运行。

2）将项目信息导入到eclipse开发工具中（branch分支）

a.pentaho项目是使用maven开发的，因此导入时使用maven的方式导入。执行菜单命令：import--existing maven projects

b.错误信息：Non-resolvable parent POM

解决方法：一般是parent标签中的relativePath配置的路径有问题。这里主要是pentaho-platform-ce-parent组件中多了parent标签，删除该标签即可。

```
<parent>
    ...
    <relativePath>../../pom.xml</relativePath>
</parent>
```

c.最后一个参数使用了不准确的变量类型的 varargs

解决方法，将maven的compile插件的showWarnings配置去掉

d.补充pom文件内容

```
<properties>
    ...
    <!-- 其他子模块的pom引用到了变量，却没定 -->
    <source.jdk.version>1.8</source.jdk.version>
    <maven-javadoc-plugin.version>2.10.4</maven-javadoc-plugin.version>
    <!-- 指定compile插件的编译版本 -->
    <java.version>1.8</java.version>
    <!-- 指定maven项目中源代码的字符集 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> 
</properties>
```

注：设置相关的变量以及

3）maven构件并运行项目

参考：https://github.com/pentaho/pentaho-platform/tree/7.1

a.构建系统运行命令

构建时可以结合mvn命令行的方式进行，同时也可以进一步排查错误

```
# 清空项目的输出信息
mvn clean
# 编译项目
mvn compile
# 打包项目，生成相关的jar以及war包
mvn clean package -Dmaven.test.skip=true
# 安装到本地
mvn clean install -DskipTests
```

注：该命令会将编辑结果存放到assemblies/pentaho-server相对目录下。

### 运行项目

在assemblies/pentaho-server/target目录下，存在pentaho-server-ce-7.1.0.8-93.zip文件，使用该文件可运行pentaho。解压文件，运行解压目录后的start-pentaho.bat文件启动服务。