---
title: gradle安装
tags: [gradle]
---

参考：https://gradle.org/install/

1）环境依赖

Gradle需要1.5或更高版本的JDK。

Gradle自带了Groovy库，所以不需要安装Groovy。并且Gradle会忽略已经安装的Groovy。

Gradle 会使用ptah中的 JDK(可以使用 java -version 检查)。

2）下载软件

下载zip包，解压后配置环境变量。将GRADLE_HOME/bin加入到你的PATH 环境变量中。

```
gradle -v
```