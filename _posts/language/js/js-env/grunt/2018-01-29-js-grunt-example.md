---
title: grunt案例
tags: [js]
---

参考：http://www.gruntjs.net/getting-started

参考2：http://blog.csdn.net/dlm_211314/article/details/48678305

创建grunt目录：Grunt项目需要两份文件：package.json和Gruntfile。

1）环境准备

系统安装了nodejs作为前提条件。安装grunt-cli

```
npm install -g grunt-cli
```

注：grunt-cli是grunt的命令行工具。另外，安装grunt-cli并不等于安装了Grunt。

2）配置环境目录

```
# 创建目录grunt
mkdir grunt
# 切换到该目录下
cd grunt
```

3）初始化package.json文件

新建并编辑package.json内容为

```
{
  "name": "my-project-name",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "~0.4.5",
    "grunt-contrib-jshint": "~0.10.0",
    "grunt-contrib-nodeunit": "~0.4.1",
    "grunt-contrib-uglify": "~0.5.0"
  }
}
```

注：在devDependencies下定义了依赖的插件信息。