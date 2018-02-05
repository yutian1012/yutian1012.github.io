---
title: grunt案例
tags: [js]
---

实战：使用grunt-contrib-uglify插件压缩项目源码文件

注：这里只压缩了以项目名为name的js文件。

参考：http://www.gruntjs.net/getting-started

参考2：http://blog.csdn.net/dlm_211314/article/details/48678305

创建grunt目录：Grunt项目需要两份文件：package.json和Gruntfile。

1）环境准备

系统安装了nodejs作为前提条件，并安装grunt-cli（运行grunt的客户端）

```
npm install -g grunt-cli
```

2）配置环境目录

```
# 创建目录grunt-example
mkdir grunt-example
# 切换到该目录下
cd grunt-example
```

3）新建并编辑package.json文件

package.json: 此文件被npm用于存储项目的元数据，以便将此项目发布为npm模块。你可以在此文件中列出项目依赖的grunt和Grunt插件，放置于devDependencies配置段内。

package.json应当放置于项目的根目录中，与Gruntfile在同一目录中，并且应该与项目的源代码一起被提交。

```
# 在grunt-example文件夹下执行命令，创建package.json
npm init -y

# 编辑package.json文件
{
  "name": "grunt-example",
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

4）新建并编辑Gruntfile.js文件

该文件用来配置或定义任务（task）并加载Grunt插件的。此文件被命名为Gruntfile.js或Gruntfile.coffee。

Gruntfile.js或Gruntfile.coffee文件是有效的JavaScript或CoffeeScript文件，应当放在你的项目根目录中，和package.json文件在同一目录层级，并和项目源码一起加入源码管理器。

```
module.exports = function(grunt) {

  // Project configuration.
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    uglify: {
      options: {
        banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
      },
      build: {
        src: 'src/<%= pkg.name %>.js',
        dest: 'build/<%= pkg.name %>.min.js'
      }
    }
  });

  // 加载包含 "uglify" 任务的插件。
  grunt.loadNpmTasks('grunt-contrib-uglify');

  // 默认被执行的任务列表。
  grunt.registerTask('default', [function(){console.log(pkg.name)},'uglify']);

};
```

5）创建项目的源代码

```
# 创建源码文件的存放目录
mkdir src
# 切换目录
cd src

# 创建一个grunt-example.js文件，文件中包含回车和缩进
function helloworld(){
    alert('helloworld');
}
```

6）安装项目所需依赖

```
# 切换到项目的根目录grunt-example下
# 运行npm install命令会将项目的依赖信息存放到该项目的node_modules文件夹中
npm install

# 运行grunt任务，下面命令都可以运行，并自动生成build文件夹，以及压缩的js文件
# 运行grunt，grunt default，grunt uglify，grunt uglify:build
grunt
```

7）压缩项目的源码文件

修改Gruntfile.js文件中uglify任务的配置信息

```
uglify: {
  options: {
    banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
  },
  build: {
        files: [{
            expand:true,    //使用占位符匹配文件名
            cwd:'src',      //js目录下
            src:'**/*.js',  //所有js文件
            dest: 'dist',   //输出到此目录下
            ext: '.min.js'  //指定扩展名
        }]
  }
}
```

或者

```
uglify: {
  options: {
    banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
  },
  build: {
    expand:true,    //使用占位符匹配文件名
    cwd:'src',      //js目录下
    src:'**/*.js',  //所有js文件
    dest: 'dist',   //输出到此目录下
    ext: '.min.js'  //指定扩展名
  }
}
```