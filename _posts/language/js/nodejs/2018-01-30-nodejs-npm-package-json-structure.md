---
title: package.json文档结构信息
tags: [nodejs]
---

参考：https://tech.jandou.com/package.json.html

参考2：http://blog.csdn.net/zmrdlb/article/details/53190696

属性详解：https://www.cnblogs.com/tzyy/p/5193811.html

### package.json文档结构

```
# 项目名称，名称中不能有空格
name: xxx

# 项目版本号
version: xxx

#项目描述
description:xxx 

# 关键词，便于用户搜索到我们的项目
keywords: [Array]

# 项目url主页
homepage: url

# 项目问题反馈的Url或email配置，json的形式提供数据值
bugs:  
{ 
    "url" : "https://github.com/owner/project/issues", 
    "email" : "project@hostname.com"
}

# 项目许可证，让使用者知道是如何被允许使用此项目。默认是"ISC"
license: xxx

# 作者和贡献者。格式设置如下： 
author,contributors: 
{ 
    "name" : "Barney Rubble",
    "email" : "b@rubble.com",
    "url" : "http://barnyrubble.tumblr.com/"
}

# 包含在项目中的文件数组。如果在数组里面声明了一个文件夹，那也会包含文件夹中的文件。
# 在项目根目录或者子目录里声明一个.npmignore，可以声明一些规则来忽略部分文件。
files: [arrays]

# 主文件。比如默认是index.js，项目名称叫mymodule。那么require(‘mymodule’)将返回index.js返回的内容
main: xxx

# 项目用到的可执行文件配置
bin: xxx

# 指定一个单一的文件名或一个文件名数组。意思类似于linux命令中的man 命令，来查看一个命令的用法 
man: xxx

# 项目代码存放地方
repository : 
{ 
    "type" : "git", 
    "url" : "https://github.com/npm/npm.git"
}

# scripts声明一系列npm脚本指令
# prepublish: 在包发布之前运行，也会在npm install安装到本地时运行
# publish,postpublish: 包被发布之后运行
# preinstall: 包被安装前运行
# install,postinstall: 包被安装后运行
# preuninstall,uninstall: 包被卸载前运行
# postuninstall: 包被卸载后运行
# preversion: bump包版本前运行
# postversion: bump包版本后运行
# pretest,test,posttest: 通过npm test命令运行
# prestop,stop,poststop: 通过npm stop命令运行
# prestart,start,poststart: 通过npm start命令运行
# prerestart,restart,postrestart: 通过npm restart运行
"scripts": {
    "start": "node index.js"
}

# 配置项目中需要的配置参数：
config : { 
    "port" : "8080"
}

# 项目在生产环境中依赖的包
"devDependencies": {
    "browserify": "~13.0.0",
    "karma-browserify": "~5.0.1"
}

# 项目在开发和测试环境中依赖的包
"devDependencies": {
    "bower": "~1.2.8",
    "grunt": "~0.4.1",
    "grunt-contrib-concat": "~0.3.0"
}

# 发布时会被一起打包的模块
bundledDependencies: [Array]

# 如果一个依赖模块可以被使用， 同时你也希望在该模块找不到或无法获取时npm继续运行，你可以把这个模块依赖放到optionalDependencies配置中。
# 这个配置的写法和dependencies的写法一样，不同的是这里边写的模块安装失败不会导致npm install失败。当然，这种模块就需要你自己在代码中处理模块的实际使用情况了
optionalDependencies: [array]

# 声明项目需要的node或npm版本范围
engines : {
    "npm" : "~1.0.20"
}

# 指定你的项目将运行在什么操作系统上
os: xxx

# 指定你的项目将运行在什么cpu架构上
cpu: xxx

# 如果你的包需要全局安装，通过命令行来运行，那么设置为true。如果这个包被本地安装则会出现一个警告
preferGlobal: true

# 如果设置为true, 那么npm会拒绝发布它
private: true

# 这个配置是会在模块发布时用到的一些值的集合，可以在这里配置tag或仓库地址。
publishConfig
```

### 配置信息的详解

1）script字段（外部可以调用该脚本）

scripts指定了运行脚本命令的npm命令行缩写，如下面指定了在scripts定义了test脚本的执行命令，那么就可以通过npm来运行。

a.创建一个HelloWorld项目

```
# 创建文件夹HelloWorld
mkdir HelloWorld
# 切换目录
# cd HelloWorld
# 新建并编辑package.json文件
notepad package.json
```

b.编辑package.json文件，在文件中指定了test脚本的运行命令

```
{
    "name": "HelloWorld",
    "version": "0.0.1",
    "scripts": {
        "test": "echo \"helloworld\""
      }
}
```

c.运行该test脚本

```
# cmd定位到package.json所在的目录下，运行命令
npm run test
```

d.运行node命令执行js文件，可以定义script

```
# 编辑package.json文件
{
    "name": "HelloWorld",
    "version": "0.0.1",
    "scripts": {
        "test": "node hello.js"
      }
}
# 新建hello.js文件
console.log("hello world!!!");
```