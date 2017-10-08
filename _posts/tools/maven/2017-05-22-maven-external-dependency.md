---
title: maven配置
tags: [maven,tools]
---

### 添加外部依赖到本地仓库

1）先将本地的jar包导入到maven本地仓库

定位ojdbc7.jar所在目录，执行安装命令

```
mvn install:install-file -Dfile=ojdbc7.jar -DgroupId=com.oracle -DartifactId=ojdbc7 -Dversion=1.0 -Dpackaging=jar
```

2）在maven中引入即可

```
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc7</artifactId>
    <version>1.0</version>
</dependency>
```

注：这里的groupId和artifactId可以任意命名。

### 引入系统中的jar包

```
<dependency>
    <groupId>json_simple</groupId>
    <artifactId>json_simple</artifactId>
    <version>1.1</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/main/webapp/WEB-INF/lib/javabuilder.jar</systemPath>
< /dependency>
```

注：scope设置为system，并在systempath路径下设置引入的jar包路径