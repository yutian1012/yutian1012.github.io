---
title: myeclipse安装groovy插件错误
tags: [groovy]
---

直接在http://marketplace.eclipse.org网站查找到groovy插件，在myeclipse安装插件时设置了下载地址。

```
groovy - http://dist.springsource.org/release/GRECLIPSE/e3.7/
```

新建groovy项目后报错

1）错误信息

安装完成groovy插件后，创建groovy project后，编译错误信息

```
Errors occurred during the build.
Errors running builder 'Java Builder' on project 'groovyJava'.
org/codehaus/jdt/groovy/integration/LanguageSupportFactory
```

2）groovy插件与eclipse的版本号是有关系的

参考：https://github.com/groovy/groovy-eclipse/wiki

3）确定出myeclipse集成的Eclipse的版本号

在myeclipse的安装目录下的readme文件夹下查看readme_eclipse.html

```
Release 4.3.0
```

4）确定出需要安装的groovy插件地址

```
groovy - http://dist.springsource.org/release/GRECLIPSE/e4.3/
```

重新安装插件，并测试。
