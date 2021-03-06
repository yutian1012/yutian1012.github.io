---
title: maven构建github上的源程序
tags: [maven]
---

在github上下载的项目源码，很多都是用maven开发的项目，而且对于历史版本中的中央仓库配置，大部分的引用都已经失效了，这样maven在运行时会有很多错误。

1）错误的忽略

这些失效的仓库可能很难被开发人员注意到，因为在开发人员本地使用的仓库基本都已经存在了，即已经下载了第三方仓库下的jar包了。而真正使用者在构建项目时会到第三方的仓库下载依赖，而第三方的仓库很可能因为某些未知的原因导致不能下载到依赖（网络不通，仓库地址发生变化等）。

2）常见的解决方法

a.使用最新的pom.xml的仓库配置覆盖本地版本的pom.xml的仓库配置

当我们将github下的项目fork到自己的github下，并选定一个版本（tag），然后checkout到自己的本地时，运行mvn命令构建项目前，可将该项目最新的pom.xml文件中的仓库配置信息覆盖掉本地项目的pom.xml的仓库配置。这样可以更新一些仓库地址。

b.对于依然不能正确下载的jar包

首先在网上查找对应的仓库，将仓库地址加到pom.xml中；其次直接下载jar文件，然后用mvn命令安装的本地仓库（mvn install）

注：关于下载jar包的问题，可以在网上下载一个官网提供的可直接运行的项目程序，然后在该项目程序下获取jar包（一般到WEB/lib下找到jar文件）。

以saiku（统计分析软件）为例，该项目托管到了github上，并且也提供了相应的运行程序，该运行程序是一个zip压缩文件，在zip压缩文件中包含了项目运行需要的所有jar，当然也存在我们需要安装到本地的jar。

3）本地编译完成后导入到eclipse中进一步运行测试。