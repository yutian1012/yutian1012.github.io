---
title: gruntfile文件
tags: [nodejs]
---

Gruntfile.js或Gruntfile.coffee文件是有效的JavaScript或CoffeeScript文件，用来配置或定义任务（task）并加载Grunt插件的。应当放在你的项目根目录中，和package.json文件在同一目录层级，并和项目源码一起加入源码管理器。

Gruntfile由以下几部分构成："wrapper"函数；项目与任务配置；加载grunt插件和任务；自定义任务。

1）案例

```
module.exports = function(grunt) {

  // Project configuration.
  grunt.initConfig({
    // 读取package.json定义的项目元数据（metadata）
    pkg: grunt.file.readJSON('package.json'),
    //配置uglify插件任务，实现将src目录下的js压缩成min.js，并输出到build目录下
    uglify: {
      options: {
        //<% %>模板字符串可以引用任意的配置属性
        // pkg.name是package.json中定义的name
        banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
      },
      //定义任务的目标
      build: {
        src: 'src/<%= pkg.name %>.js',
        dest: 'build/<%= pkg.name %>.min.js'
      }
    }
  });

  // 加载包含 "uglify" 任务的插件。
  grunt.loadNpmTasks('grunt-contrib-uglify');

  // 默认被执行的任务列表，注册上面定义的uglify的build
  grunt.registerTask('default', ['uglify:build']);

};
```

注：grunt-contrib-uglify

2）grunt-contrib-uglify插件

uglify是一个文件压缩插件，用来将文本文件进行压缩存储。

### Gruntfile结构信息

1）wrapper函数

每一份Gruntfile（和grunt插件）都遵循同样的格式，你所书写的Grunt代码必须放在此函数内。

```
module.exports = function(grunt) {
  // Do grunt-related things in here
};
```

2）项目和任务配置

大部分的Grunt任务都依赖某些配置数据，这些数据被定义在一个object内（js的object对象），并传递给grunt.initConfig方法。

```
grunt.initConfig({
  //将存储在package.json文件中的JSON元数据引入到grunt config中
  pkg: grunt.file.readJSON('package.json'),
  uglify: {
    options: {
      //<% %>模板字符串可以引用任意的配置属性
      banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
    },
    //定义目标
    build: {
      src: 'src/<%= pkg.name %>.js',
      dest: 'build/<%= pkg.name %>.min.js'
    }
  }
});
```

3）加载 Grunt 插件和任务

像concatenation、minification、grunt-contrib-uglify和linting这些常用的任务（task）都已经以grunt插件的形式被开发出来了。只要在package.json文件中被列为dependency（依赖）的包，并通过npm-install安装之后，都可以在Gruntfile中以简单命令的形式使用

```
// 加载能够提供"uglify"任务的插件。
grunt.loadNpmTasks('grunt-contrib-uglify');
```

4）自定义任务

通过定义default任务，可以让Grunt默认执行一个或多个任务。default任务列表数组中可以指定任意数目的任务（可以带参数）。

```
//执行grunt命令时如果不指定一个任务的话，将会执行uglify任务。
//这和执行grunt uglify 或者 grunt default的效果一样。

grunt.registerTask('default', ['uglify']);
```

如果Grunt插件中的任务（task）不能满足你的项目需求，你还可以在Gruntfile中自定义任务（task）。

```
module.exports = function(grunt) {

  //注册custom任务
  grunt.registerTask('custom', 'testfunction', function() {
    grunt.log.write('Logging some stuff...').ok();
  });
};

# 运行时执行
grunt custom
```

5）实战，运行自定义任务

```
# 创建目录
mkdir test
# 切换目录
cd test
# 创建package.json文件
npm init -y
# 编辑package.json文件
{
  "name": "test",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "~0.4.5"
  }
}
# 创建Gruntfile.js文件
notepad Gruntfile.js
# 编辑文件
module.exports = function(grunt) {
  //注册custom任务
  grunt.registerTask('custom', 'testfunction', function() {
    grunt.log.write('Logging some stuff...').ok();
  });
};
# 安装grunt项目依赖，会生成node_modules文件夹，存放依赖
npm install
# 运行自定义任务
grunt custom
```