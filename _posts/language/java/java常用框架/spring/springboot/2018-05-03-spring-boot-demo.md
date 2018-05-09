---
title: spring boot入门
tags: [spring]
---

1）快速入门（在线生成工具）

使用在线的工具生成一个springboot的maven项目，可以自定义添加的依赖包。

选择Maven project，java（语言），以及springboot的版本号。通过设置Group和Artifact设置maven项目的属性信息，通过Dependencies设置依赖的jar文件。

在线工具：http://start.spring.io/

注：该地址提供了一个springboot项目的初始化工具，可以生成相应的压缩包，然后下载。下载后解压文件，是一个maven项目。

2）生成的文件目录

```
│  .gitignore
│  mvnw
│  mvnw.cmd
│  pom.xml
├─.mvn
│  └─wrapper
│          maven-wrapper.jar
│          maven-wrapper.properties
│
├─src
│  ├─main
│  │  ├─java
│  │  └─resources
│  │
│  └─test
│      └─java
```

a.mvnw.cmd文件

该文件是一个maven-wrapper-script,它可以让你在没有安装maven或者maven版本不兼容的条件下运行maven的命令。

b.application.properties文件

在src/main/resource目录下存在applicant.properties文件，该文件是用来进行相关配置的文件，如数据库配置等。

3）常见问题

将下载的maven项目导入到eclipse中的错误提示

```
Syntax error, annotations are only available if source level is 1.5 or greater
```

设置编译版本

```
<!-- 指定一下jdk的版本 ，这里我们使用jdk 1.8，但在myeclipse中会有问题-->
<java.version>1.8</java.version>

<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.7</source>
        <target>1.7</target>
    </configuration>
</plugin>
```

注：这里由于myeclipse最大支持1.7，所以编译时只能先指定1.7，指定为1.8会报错。

4）查看pom.xml文件

springboot的依赖，由于我们使用在线工具创建maven项目时，并没有添加任何的依赖。因此，pom.xml文件中会将springboot的最核心依赖添加到项目中。

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath/>
</parent>

...

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

a.spring-boot-starter-parent

b.spring-boot-starter

核心模块，包括自动配置支持、日志和YAML。

c.spring-boot-starter-test

测试模块，包括JUnit、Hamcrest、Mockito

5）运行测试（三种方式运行）

a.IDE环境下运行DemoApplication

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

注：该类是由在线生成工具自动生成的。

错误提示：

```
Unsupported major.minor version 52.0
```

错误原因是，java代码在编译时使用的是1.8版本的编译器（引入的依赖jar包的类可能是使用1.8编译的jar文件），而Myeclipse使用的版本IDE目前只能支持到1.7版本的编译。因此会报版本不匹配问题。

解决方法：右击项目--properties--java build path选择jre的libraries为1.8，那么在IDE运行环境中，就会使用1.8版本的环境进行运行。

![](/images/spring/springboot/initializer-ide-jre.png)

b.命令行方式运行

这种方式运行使用spring-boot-loader-tools等工具用来加载并运行。

```
mvn clean compile

mvn spring-boot:run
```

错误提示：

```
Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.7.0:compile
无效的标记: -parameters
```

问题：本地运行java的环境是1.7，而实际要求的环境是1.8

```
# 确定当前环境中使用的java运行环境
echo %JAVA_HOME%
```

修改方法：将java运行环境切换成1.8即可。

c.生成jar后运行

```
mvn clean package -Dmaven.test.skip=true

cd target

java -jar demo-0.0.1-SNAPSHOT.jar
```