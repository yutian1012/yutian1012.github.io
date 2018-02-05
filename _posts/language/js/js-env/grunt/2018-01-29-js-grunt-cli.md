---
title: grunt-cli
tags: [js]
---

参考：http://www.gruntjs.net/getting-started

grunt-cli是运行grunt的客户端或命令行环境。

安装grunt-cli并不等于安装了Grunt！Grunt-CLI的任务很简单：调用与Gruntfile在同一目录中 Grunt。这样带来的好处是，允许你在同一个系统上同时安装多个版本的Grunt。这样就能让多个版本的 Grunt 同时安装在同一台机器上。

1）安装grunt-cli

```
npm install -g grunt-cli
```

注：grunt-cli是grunt的命令行工具。另外，安装grunt-cli并不等于安装了Grunt。

2）安装位置

```
# windows系统安装完成后的路径，userxxx表示windows下的用户
C:\Users\userxxx\AppData\Roaming\npm
```

注：并且安装完成后会将该目录设置到环境变量path中。

3）查看安装的grunt-cli的版本

```
grunt --version
```

4）grunt-cli工作原理

每次运行grunt时，他就利用node提供的require系统查找本地安装的Grunt。正是由于这一机制，你可以在项目的任意子目录中运行grunt。如果找到一份本地安装的Grunt，CLI就将其加载，并传递Gruntfile中的配置信息，然后执行你所指定的任务。

注：grunt-cli源码中使用require函数查找grunt。