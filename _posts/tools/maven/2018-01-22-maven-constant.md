---
title: maven 中的变量与常量信息
tags: [maven,tools]
---

参考：http://blog.csdn.net/zheng12tian/article/details/40617421

1）变量设置

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

2）内置常量

Maven内置变量说明：

```
# 项目根目录
${basedir} 
# 构建目录，缺省为target
${project.build.directory} 
# 构建过程输出目录，缺省为target/classes
${project.build.outputDirectory} 
# 生成的jar或war名称，缺省为${project.artifactId}-${project.version}
${project.build.finalName} 
# 打包类型，缺省为jar
${project.packaging} 
# 当前pom文件的任意节点的内容。如${project.version}获取定义的版本号
${project.xxx} 
```