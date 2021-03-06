---
title: 持续集成jenkins实例
tags: [jenkins]
---

参考：https://jenkins.io/doc/tutorials/build-a-java-app-with-maven/

1）创建一个maven项目HelloWorld程序

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.ipph.test</groupId>
  <artifactId>helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>helloworld</name>
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>com.github.stefanbirkner</groupId>
      <artifactId>system-rules</artifactId>
      <version>1.16.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>  
      <plugins>  
        <plugin>  
          <groupId>org.apache.maven.plugins</groupId>  
          <artifactId>maven-compiler-plugin</artifactId>  
          <configuration>  
            <source>1.7</source>  
            <target>1.7</target>  
            <encoding>UTF-8</encoding>  
          </configuration>  
        </plugin>  
      </plugins>  
    </build>  
</project>
```

2）添加一个待编译的程序

```
package org.ipph;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

3）提交代码到svn仓库中

首先在SVNServer上创建仓库helloworld，在项目上右击team--shared project。

4）jenkins创建配置任务

第一步：创建一个新任务--输入任务名称（helloworld），并选择构建一个自由风格的软件项目。自由风格的软件项目具有更大的灵活性，可以把构建作业分成若干步骤。

![](/images/tools/jenkins/create-job-type.png)

第二步：配置项目信息，配置源代码仓库地址

![](/images/tools/jenkins/create-job-scm.png)

第三步：配置构建触发器

```
选择Poll SCM，即当代码库有提交时触发，

设置日程表为*/3 * * * *（每3分钟检测一下代码库是否有提交）。
```

![](/images/tools/jenkins/create-job-trigger.png)

第四步：配置构建环境

点击添加构建步骤按钮，选择invoke top-level maven targets选项，可用来设置maven命令完成该步骤的工作。

```
# 设置mvn执行命令clean package，并且前面不要添加mvn命令
clean package
```

![](/images/tools/jenkins/create-job-build-command.png)

第五步：生成测试报告

点击增加构建后操作按钮（在构建之后生成测试报告），选择publish junit test result report选项，并配置junit生成测试报告xml文档路径。所有的junit测试框架都会生成xml文档，用来反馈测试结果，jenkins能够识别xml文档内容，并对其进行分析，进而展示报表，测试结果统计等。

这里项目中使用的是junit，要清楚mvn运行测试后生成的测试xml文档位置，然后在配置该路径，如mvn运行test命令后，会将结果生成到target\surefire-reports目录下，文档为xml类型。

```
# jenkins会在构建的项目根目录下查找生成的测试文件
设置路径为：target\surefire-reports\*.xml
# 更加健壮的配置方式，开头的两个*号能够匹配多路径层次
**/target/surefire-reports/*.xml
```

![](/images/tools/jenkins/create-job-test-report.png)

第六步：将构建产物进行归档

如应用程序一般会打包成jar，web程序会打包成war，然后部署到相应的环境中运行。这里jenkins也提供了对构建产物进行归档的操作。

点击增加构建后操作按钮，选择archive the artifacts，并设置存档路径。

```
# 设置构建产物存档路径
**/target/*.jar
```

![](/images/tools/jenkins/create-job-archive.png)

5）新建测试类

```
package org.ipph.test.helloworld;

import static org.junit.Assert.assertTrue;

import org.ipph.HelloWorld;
import org.junit.Rule;
import org.junit.Test;
import org.junit.contrib.java.lang.system.SystemOutRule;

/**
 * Unit test for simple App.
 */
public class AppTest 
{
  @Rule
    public final SystemOutRule log = new SystemOutRule().enableLog();
    /**
     * Rigorous Test :-)
     */
  @Test
    public void testOutput()
    {
      log.clearLog();
      
        HelloWorld.main(null);
        
        assertTrue(log.getLog().contains("Hello World"));
    }
}
```

注：svn将提交代码到仓库，当jenkins检测到有代码提交（定时任务）时，便会开始构建运行指定的mvn命令。

6）管理页面的操作

a.查看任务列表（主页面）

![](/images/tools/jenkins/admin-job-list.png)

b.查看任务明显（点击列表中的任务名称）

任务明显中提供了删除，编辑，构建历史，测试报告等信息。

![](/images/tools/jenkins/job-details.png)

c.查看构建输出信息

![](/images/tools/jenkins/create-job-buildresult.png)

注：构建的过程文件保存在.jenkins\jobs\helloworld\builds文件夹下

7）错误信息

```
ERROR: Step ‘Publish JUnit test result report’ aborted due to exception:
Nested exception: 前言中不允许有内容
```

原因是测试输入的xml文档路径指定错误，需要根据实际mvn运行后测试结果的输出位置指定相应的xml文件。