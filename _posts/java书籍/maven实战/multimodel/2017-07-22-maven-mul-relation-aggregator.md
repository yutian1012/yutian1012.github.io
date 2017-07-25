---
title: maven多模块关联--聚合
tags: [maven]
---

项目场景（简单的账号注册服务）：用户注册后，通过提供的邮箱进行激活，并登录。

注：参考第4章的项目信息介绍

### 介绍

多模块间主要存在模块之间的继承关系和依赖关系（参考maven实战的第8章）。该背景案例最终会被划分为account-email，account-persist等五个模块。Maven的聚合特性能够把项目的各个模块聚合在一起构建，而Maven的继承特性则能帮助抽取各个模块相同的依赖和插件等配置，简化并规范POM，促进了各个模块的一致性。

### 模块间的聚合

在一个项目下可能会存在多个子模块（子项目），在构建时，我们会想要以次构建多个子项目，而不是到多个子项目下分别执行mvn命令构建。

为了能够使用一条命令就能够构建account-email和account-persist两个模块，我们需要创建一个额外的名为account-aggregator的模块，然后通过该模块构建整个项目的所有模块。account-aggregator本身作为一个Maven项目。

1）新建聚合的maven项目（平行关系）

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Aggregator</name>
    <modules>
        <module>../account-email</module>
        <module>../account-persist</module>
    </modules>
</project>
```

对于聚合模块来说，其打包方式packaging的值必须为pom，否则就无法构建。

modules元素是聚合的最核心配置，每个module的值都是一个当前pom的相对目录。

![](images/book/maven/ch8/aggregator-parall.png)

2）聚合maven模块作为顶层模块（父子关系）

为了方便用户构建项目，通常将聚合模块放在项目目录的最顶层，其他模块作为聚合模块的子目录存在。因此，当用户得到源码的时候，第一眼发现的就是聚合模块的pom，不用从多个模块中去寻找聚合模块来构建整个项目。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Aggregator</name>
    <modules>
        <module>account-email</module>
        <module>account-persist</module>
    </modules>
</project>
```

注：这里module的值为account-email，因为该文件夹与pom.xml文件处于相同的目录下

![](images/book/maven/ch8/aggregator-top.png)

### eclipse下聚合模块中创建子模块

1）创建子模块

在maven聚合模块上右击--new--maven module

![](/images/book/maven/ch8/new_maven_module.png)

![](/images/book/maven/ch8/new_maven_module2.png)

![](/images/book/maven/ch8/new_maven_module3.png)

![](/images/book/maven/ch8/new_maven_module4.png)

注：groupId和version都会从聚合模块中获取。

2）创建完后的视图

创建完成后子模块会出现在聚合模块下，并且在eclipse中会出现一个单独的项目account-test。

![](/images/book/maven/ch8/new_maven_module5.png)

3）maven project和module的区别

Maven Project可以理解为父工程，Maven Module可以理解为子工程。创建Maven Module工程必须有存在的父工程。