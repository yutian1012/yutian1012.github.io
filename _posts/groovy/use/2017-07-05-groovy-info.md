---
title: groovy简介
tags: [groovy]
---

参考：http://www.groovy-lang.org/documentation.html

### 介绍

Groovy是一种运行在JVM上的动态语言，它吸取了Python，Ruby和Smalltalk等语言的优点，在Java语言的基础之上增加了许多特色功能。对于Java开发人员来说掌握Groovy是没有什么太大障碍的。相比Java而言，语法更易懂，更易上手，更易调试。无缝的集成了Java 的类和库。编译后的.groovy也是以class的形式出现的。

### 下载安装

参考：http://www.groovy-lang.org/install.html

1）下载

http://www.groovy-lang.org/download.html

下载方式提供了binary，source，windows installer等。

注：当前使用的版本为groovy2.4需要运行在jdk1.7+的环境上。

2）配置环境变量

设置GROOVY_HOME，path中添加%GROOVY_HOME%/bin

查看安装，运行命令

```
groovy -version
```

![](/images/groovy/info/groovy-version.png)

### HelloWorld实例

1）了解groovy的shell

在cmd中打开groovy shell

```
groovysh
```

![](/images/groovy/info/groovy-shell.png)

注：按ctrl+d退出脚本命令行。

2）新建groovy脚本文件

在D盘下新建helloworld.groovy文件，并编辑内容

```
println "helloworld!";
```

运行脚本

```
groovy D:/helloworld.groovy
```

![](/images/groovy/info/groovy-helloworld.png)

### eclipse下安装groovy插件

1）安装插件

在http://marketplace.eclipse.org/中查找groovy插件并安装。

2）新建groovy project项目(项目名groovyJava)

![](/images/groovy/info/newgroovyproject.png)

3）添加junit测试

![](/images/groovy/info/newgroovyprojectjunit.png)

4）新建Hellowold类并测试

```
class Helloworld {
    @Test
    void testHelloworld(){
        println "helloworld";
    }
}
```

![](/images/groovy/info/groovy-helloworld2.png)
