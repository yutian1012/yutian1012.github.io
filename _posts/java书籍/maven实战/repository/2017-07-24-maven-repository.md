---
title: maven仓库分类
tags: [maven]
---

在maven世界中，任何一个依赖，插件或者项目构建的输出，都可以称为构件。maven仓库就是要用来管理这些构件的。

### 仓库的分类

1）可将maven仓库分为两类：本地仓库和远程仓库。

注：用户能够使用的构件一定会在本地仓库中，即只有本地仓库中的构件才能被maven项目直接使用，否则就有从远处下载构件，放到本地仓库，然后再使用。

![](/images/book/maven/repository/repository-category.png)

2）本地仓库

当maven需要使用依赖文件时，总是基于坐标使用本地仓库的依赖文件。每个用户在自己的用户目录下都有一个路径名为.m2/repository/的仓库目录。

注：一个构件只有在本地仓库中之后，才能由其他maven项目使用。

本地仓库的配置文件~/.m2/settings.xml，该文件一般是不存在的，需要复制%MVN_HOME%/conf/setings.xml。这样不同用户就可以自定义自己的maven配置，避免无意识的影响到系统中的其他用户。推荐不要直接修改全局目录下的settings.xml文件（会造成升级maven不得不重新修改配置）。

设置仓库位置，在settings.xml中设置

```
<settings>
    <localRepository>D:\java\repository</localRepository>
</settings>
```

安装maven项目到本地仓库命令：

```
mvn clean install
```

注：使用install命令能够将maven项目构件安装到本地仓库中，供其他maven项目使用。

3）中央仓库

中央仓库是一个特殊的远程仓库，是maven核心自带的远程仓库。它包含了绝大部分开源的构件。在默认配置下，当本地仓库没有maven需要的构件时，就会尝试从中央仓库下载。

Maven的安装文件自带了中央仓库的配置，可以使用解压工具打开jar文件（%MVN_HOME%/lib/maven-model-builder-3.0.jar），然后访问路径org/apache/maven/model/pom-4.0.0.xml文件（超级POM）。

配置信息

```
<repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
</repositories>
```

注：所以maven项目都会继承超级POM。这段配置使用id为central对中央仓库进行唯一标识。

4）私服

私服是另外一种特殊的远程仓库，为了节省带宽和时间（加速Maven构建），应该在局域网内架设一个私有的仓库服务器。用其代理所有外部的远程仓库，即私服代理广域网上的远程仓库，供局域网的maven用户使用。内部的项目还能部署到私服上供其他项目使用。另外，一些无法从外部仓库下载到的构件也能从本地到私服上供大家使用。

![](/images/book/maven/repository/repository-owner.png)

私服的作用：节省外网带宽，加速Maven构建，补上第三方构件，提高稳定性，增强控制，降低中央仓库的负荷。