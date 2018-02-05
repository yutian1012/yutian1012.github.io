---
title: npm包项目的配置信息
tags: [nodejs]
---

### package.json文件

package.json: 此文件被npm用于存储项目的元数据，以便将此项目发布为npm模块。你可以在此文件中列出项目依赖的插件，放置于devDependencies配置段内。同时该文件定义了项目的基本信息等。

package.json文件内部就是一个JSON对象，该对象的每一个成员就是当前项目的一项设置。

package.json应当放置于项目的根目录中，与Gruntfile在同一目录中，并且应该与项目的源代码一起被提交。

注：很多属性可以通过npm-config来生成

1）文件的生成

a.可以手动创建并编辑package.json文档。

b.通过npm-init命令生成package.json文档。

```
# 根据系统提示信息设置相关字段，最后生成package.json
git init
# 默认值方式生成package.json
npm init –yes|-y
```

在你要创建的目录下执行：npm init，系统会一一提示设置相关配置。提示设置的字段均为必填字段（有的可以用回车键，即设置为空带过）。npm init –yes|-y: 执行此命令，则会直接创建一个package.json，只配置了一些必填字段，并且给出默认值。其中name: 所处的文件夹名称。

注：可以再生成后对package.json文件进行修改。

2）文件的作用

该文件即为打包需要，同时又为安装包所需要。

npm install npm包，命令根据这个配置文件，自动下载所需的模块，也就是配置项目所需的运行和开发环境，以便该软件包能够正常运行，或引入该插件的环境能够运行该插件。

3）完整的文档实例

```
{
    "name": "Hello World",
    "version": "0.0.1",
    "author": "张三",
    "description": "第一个node.js程序",
    "keywords":["node.js","javascript"],
    "repository": {
        "type": "git",
        "url": "https://path/to/url"
    },
    "license":"MIT",
    "engines": {"node": "0.10.x"},
    "bugs":{"url":"http://path/to/bug","email":"bug@example.com"},
    "contributors":[{"name":"李四","email":"lisi@example.com"}],
    "scripts": {
        "start": "node index.js"
    },
    "dependencies": {
        "express": "latest",
        "mongoose": "~3.8.3",
        "handlebars-runtime": "~1.0.12",
        "express3-handlebars": "~0.5.0",
        "MD5": "~1.2.0"
    },
    "devDependencies": {
        "bower": "~1.2.8",
        "grunt": "~0.4.1",
        "grunt-contrib-concat": "~0.3.0",
        "grunt-contrib-jshint": "~0.7.2",
        "grunt-contrib-uglify": "~0.2.7",
        "grunt-contrib-clean": "~0.5.0",
        "browserify": "2.36.1",
        "grunt-browserify": "~1.3.0",
    }
}
```