---
title: maven继承
tags: [maven,tools]
---

参考：http://www.cnblogs.com/AlanLee/p/6211785.html

### maven继承

1）介绍

maven如果多个模块出现相同的依赖包，这样在pom.xml文件的内容出现了冗余、重复的内容，解决这个问题其实使用Maven的继承机制即可，就像Java的继承一样，父类就像一个模板，子类继承自父类，那么有些通用的方法、变量都不必在子类中再重复声明了。Maven的继承机制类似，在一个父级别的Maven的pom文件中定义了相关的常量、依赖、插件等等配置后，实际项目模块可以继承此父项目 的pom文件，重复的项不必显示的再声明一遍了，相当于父Maven项目就是个模板，等着其他子模块去继承。不过父Maven项目要高度抽象，高度提取公共的部分（交集），做到一处声明，多处使用。

### maven继承实例

1）抽象出共性作为父模块

创建parent模块，然后通过该模块来作为所有模块的父POM。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 如果groupId不需要特别指明的话，可以继承父POM定义的值 -->
    <!--<groupId>xxx</groupId>-->
    <artifactId>xxx</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>xxxParent</name>
    <url>http://maven.apache.org</url>

    <!-- 抽取出其他子模块中需要使用的变量信息 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven-compiler-plugin.version>3.1</maven-compiler-plugin.version>
        <mysql-driver.version>5.1.25</mysql-driver.version>
        ...
    </properties>

    <!-- 抽取出其他子模块中需要依赖的jar包 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql-driver.version}</version>
            </dependency>
            ...
        </dependencies>
    </dependencyManagement>    

    <!-- 定义模块需要发布的路径信息  -->
    <distributionManagement>
        <repository>
            <id>releases</id>
            <name>public</name>
            <url>http://59.50.95.66:8081/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>Snapshots</name>
            <url>http://59.50.95.66:8081/nexus/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>

    <!-- 定义编译时需要使用的插件 -->
    <build>
        <pluginManagement>  
            <plugins>  
                <plugin>  
                    <groupId>org.apache.maven.plugins</groupId>  
                    <artifactId>maven-compiler-plugin</artifactId>  
                    <version>3.1</version>  
                    <configuration>  
                        <source>1.7</source>  
                        <target>1.7</target>  
                    </configuration>  
                </plugin>
                ...  
            </plugins>  
        </pluginManagement>
    </build>

    <!-- 定义插件仓库地址 -->
    <pluginRepositories>
        <pluginRepository>
            <id>nexus</id>
            <name>nexus</name>
            <url>http://192.168.0.70:8081/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

</project>
```

注：在父POM中配置项目中定义依赖使用dependencyManagement标签，并不会将相关的依赖引入到子模块中。子模块需要单独引入。即把依赖放入dependencyManagement元素当中，这样的依赖就成了可选的。

注2：插件的继承与依赖的继承是类似的。

2）子模块继承父模块

子模块会自动继承父模块定义的properties信息，以及仓库配置信息等。但对于jar包依赖和插件依赖需要子模块单独引入。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

    <!-- 引入父模块 -->
    <!-- relativePath元素是可选的，默认值为../pom.xml-->
    <parent>
        <groupId>com.uidp</groupId>
        <artifactId>UidpParent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <groupId>xxxsub</groupId>
    <artifactId>xxx</artifactId>
    <packaging>war</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>xxx</name>
    <url>http://maven.apache.org</url>

    <!-- 子模块中引入jar，如果在父模块中配置了依赖，可省略版本号 -->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        ....
    </dependencies>

    <!-- 引入插件-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
            ...
        </plugins>
    </build>

</project>
```

注：当项目构建时，Maven会首先根据relativePath检查父POM，如果找不到，再从本地仓库找。