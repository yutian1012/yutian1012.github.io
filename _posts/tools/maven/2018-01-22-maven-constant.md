---
title: maven 中的变量与常量信息
tags: [maven,tools]
---

参考：http://blog.csdn.net/hongweigg/article/details/54091148

1）变量设置/自定义属性（properties标签中定义）

maven使用时通常会用到一些Maven变量，因此需要找个地方对这些变量进行统一定义。在根节点project下增加properties节点，所有自定义变量均可以定义在此节点内。

```
<!-- 全局属性配置 -->  
<properties>  
    <spring.version>4.0.5.RELEASE</spring.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
</properties>  
```

变量的使用，使用${}来引用定义的变量信息

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
</dependency>
```

2）内置常量（Maven预定义,用户可以直接使用）

Maven内置变量说明：

```
# 表示项目根目录,即包含pom.xml文件的目录;
${basedir}
# 表示项目版本;
${version}
# 同${basedir};
${project.basedir}
# 表示项目文件地址;
${project.baseUri}
# 表示项目构件开始时间;
${maven.build.timestamp}
# 表示属性${maven.build.timestamp}的展示格式，默认值为yyyyMMdd-HHmm
# 可在pom的properties标签中配置输出格式，如配置格式为
# <properties>
#    <maven.build.timestamp.format>
#        yyyy-MM-dd HH:mm:ss
#    </maven.build.timestamp.format>
# </properties>
${maven.build.timestamp.format}
```

3）pom属性（需要在pom中指定标签值）

使用pom属性可以引用到pom.xml文件对应元素的值，即在pom中定义的元素。

```
# 表示主源码路径;<build><directory>xxx</directory></build>
${project.build.directory}
# 表示主源码的编码格式;
${project.build.sourceEncoding}
# 表示主源码路径;
${project.build.sourceDirectory}
# 表示输出文件名称;
${project.build.finalName}
# 表示项目版本,与${version}相同;
${project.version}
```

注：前提是要在pom中使用相应的标签。

4）settings.xml文件属性

与pom属性同理,用户使用以settings.开头的属性引用settings.xml文件中的XML元素值。

```
# 表示本地仓库的地址;
${settings.localRepository}
```

5）Java系统属性(所有的Java系统属性都可以使用Maven属性引用)

```
# 使用命令可查看所有的Java系统属性;
mvn help:system
# Java代码运行可得到所有的Java属性;
System.getProperties()
# 表示用户目录;
${user.home}
```

6）环境变量属性

所有的环境变量都可以用以env.开头的Maven属性引用。

```
# 使用命令可查看所有环境变量;
mvn help:system
# 表示JAVA_HOME环境变量的值;
${env.JAVA_HOME}
```