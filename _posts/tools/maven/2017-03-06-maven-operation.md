---
title: maven常用操作
tags: [maven,tools]
---

### 1. 导出项目源码包

```
mvn source:jar
```

注：会在项目的target下生成以-sources.jar为后缀的源码的jar包。

### 2. 打包项目

```
mvn package -Dmaven.test.skip=true
```

注：会在项目的target目录下生成相应的jar或war包。

### 3. 编译项目

```
mvn compile
```

注：会在target目录下生成一列的classes目录。

### 4. 下载依赖

```
mvn dependency:copy-dependencies
```

注：会在target目录下生成dependency目录，目录下放置项目依赖的所有jar文件。

### 5. 下载依赖源码包

```
mvn dependency:sources
```

注：关联依赖源码后，可以在IDE工具中方便查看。

### 6. 查看项目依赖

1）list命令

```
mvn dependency:list
```

2）tree命令

```
mvn dependency:tree
```

