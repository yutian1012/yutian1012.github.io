---
title: spring源码查看和运行
tags: [spring]
---

参考：http://blog.csdn.net/paincupid/article/details/43902437

官网参考源码下载（readme文件）：https://github.com/spring-projects/spring-framework/tree/v4.3.0.RC2

spring源码的托管地址

```
https://github.com/spring-projects/spring-framework
```

### 环境准备

1）下载并安装jdk8（设置环境变量JAVA_HOME），git工具，以及gradle（GRADLE_HOME）。

2）运行git的clone命令将项目下载到本地

```
git clone git@github.com:spring-projects/spring-framework.git
```

注:运行完成后会在相应的文件夹中生成spring-framework该文件夹中存放spring的所有组件的源码。

3）引入该文件夹，并执行命令

在cmd中执行import-into-eclipse.bat命令

注：如果系统中没有安装gradle，甚至会自动下载并安装。

gradle的下载操作可查看gradlew.bat批处理文件。会运行gradle\wrapper\gradle-wrapper.jar中的org.gradle.wrapper.GradleWrapperMain类，该类中应该包含下载并安装gradle的实际代码。