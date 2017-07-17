---
title: java源码查看--eclipse为jar包添加源码查看
tags: [basic]
---

### 针对maven项目添加源码

```
mvn dependency:sources
```

首先运行命令下载源码包后，在项目中进行关联即可查看。

### 关联项目中的lar包源码查看

在项目所在的lib包上右击，选择properties--java source attachment

![](/images/java_basic/sorcecode/addsource.png)

选择下载好的相关依赖信息

![](/images/java_basic/sorcecode/sourceAttachment.png)

注：针对jdk一般都会在安装目录下提供一个src.zip压缩文件，该文件就是对应的源文件压缩包。

注2：针对一些第三方的jar包，一般官网或maven仓库中存在相应的源码，可自行下载手动关联查看。