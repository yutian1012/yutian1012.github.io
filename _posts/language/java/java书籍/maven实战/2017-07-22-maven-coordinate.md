---
title: maven坐标
tags: [maven]
---

就像Make的Makefile，ant的build.xml一样，maven项目的核心是pom.xml。该文件定义了项目的基本信息，用于描述项目如何构建，声明依赖等。

### maven坐标

pom.xml文件中定义了groupId，artifactId和version，这三个元素定义了项目的基本坐标。在maven中任何的jar，pom或者war都是以这些基本的坐标进行区分的。

1）groupId元素

定义了项目属于哪个组，这个组往往和项目所在的组织或公司存在关联。如googlecode上建立了一个名为myapp的项目，那么groupId就应该是com.googlecode.myapp（com表示company，或者使用org表示organization，googlecode表示公司名称，myapp表示项目名称）。

2）artifactId元素

定义了当前Maven项目在组中唯一的ID，目的是定义项目的组件，一个项目可能分解成多个组件，artifactId表示项目中的组件。如前面的groupId为com.googlecode.myapp的例子中，你可能会为不同的子项目（模块）分配artifactId，如myapp-util，myapp-domain，myapp-web等。

注：子项目的artifactId的前缀一般使用项目名，以方便同其他项目区分。

3）version元素

制定了当前项目的版本信息，如1.0-SNAPSHOT。SNAPSHOT表示快照，说明该项目还处于开发中，是不稳定的版本。随着项目的发展，version会不断更新，如升级为1.0、1.1-SNAPSHOT、1.1、2.0等。（参考maven管理项目版本的升级发布--第13章）

### 实例

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.google.myapp</groupId>
  <artifactId>myapp-util</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>myapp-util</name>
  <url>http://maven.apache.org</url>
</project>
```

modelVersion指定了当前pom模型的版本，对于maven2及maven3来说，它的值只能是4.0.0

package表示构建打包时的格式，如jar或war等

name元素声明了一个对于用户更为友好的项目名称，方便信息交流。