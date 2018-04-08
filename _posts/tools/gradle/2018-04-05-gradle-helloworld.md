---
title: gradle构建入门实例
tags: [gradle]
---

1）创建hello文件夹

该文件夹作为第一个项目的入口文件夹。所有的操作都在该目录下进行

```
E:\work\test\gradle\hello
```

2）创建构建脚本build.gradle

build.gradle文件为构建脚本，可在该构建脚本定义了一个project和一些默认的task。

```
task hello {
    doLast {
        println 'Hello world!'
    }
}
```

上面的脚本定义了一个叫做hello的task，并且给它添加了一个动作。当执行gradle hello的时候，Gralde便会去调用hello这个任务来执行给定操作。这些操作其实就是一个用groovy书写的闭包。

doLast函数将需要最后执行的Action放入task队列中，Action就是一个闭包（即这里的{xxx}代码）。

注：Gradle的tasks就相当于Ant中的targets，只是换了一个比target更形象的另外一个术语。

3）执行命令

```
gradle -q hello
```

注：-q参数用来控制gradle的日志级别，可以保证只输出我们需要的内容。

4）快速构建任务语法

```
task hello << {
    println 'Hello world!'
}
```

该方法与上面的方式相同，<< {xxx}的时候，创建了一个Task对象，同时把closure做为一个action加到这个Task的action队列中，并且告诉它“最后才执行这个closure”（注意，<<符号是doLast的代表）。