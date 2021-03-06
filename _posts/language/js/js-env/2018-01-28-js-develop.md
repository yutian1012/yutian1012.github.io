---
title: js开发，调试环境
tags: [js]
---

### 工具集

前端工具基本是以nodejs作为底层，即运行时需要nodejs提供的环境。使用这些工具集的组合，能够实现自动化的js测试环境。

![](/images/js/env/tools.png)

### 1、代码编辑工具

支持语法检查，整合调试。

a.推荐使用sublime。

sublime中可以集成多个插件，简化开发。如Emmet（zencoding）插件能够是html页面编码更快捷方便。

b.webstorm工具

该工具是收费软件。功能强大，提供了类似java IDE的开发环境。

### 2、断点调试工具

a.batarang

Batarang是AngularJS调试的Google-chrome插件，安装后可以在google浏览器中对AngularJS应用代码进行调试。

注：batarang适用于angular程序的调试，很多菜单是专门针对angularJs进行设计的。

b.firebug

firebug该调试工具更通用。

### 3、版本管理工具

使用git作为js代码版本工具，安装完成后只能使用命令行的方式运行。如果想要使用图形化界面的方式，可以安装tortoisegit。

### 4、代码合并和混淆工具

防止别人窃取程序实现逻辑，防止暴露js源代码。grunt是用来进行代码混淆的工具，该工具需要依赖nodejs。可以使用nodejs的npm包管理器进行包下载。

1）grunt插件的作用

a.js文件合并

b.JS代码自动压缩

注：每次Ctrl+S的时候自动执行以上动作。另外，也可设置在每次ctrl+s自动运行单元测试和集成测试。

2）grunt插件

参考：http://www.gruntjs.net

grunt插件在使用时一般会结合grunt提供的5个插件配合使用。grunt-contrib-uglify（对代码进行混淆）、grunt-contrib-qunit、grunt-contrib-concat（合并文件）、grunt-contrib-jshint、grunt-contrib-watch（监控文件变化）。

![](/images/js/env/grunt/grunt-plugin.png)

### 依赖管理工具

防止js插件的冲突，在项目很大时，需要引入很多的js插件，导入很多开源的控件。尤其当这些插件版本可能存在冲突的时候，如果完全人工维护，显示效率太低。

### 单元测试工具

前端的单元测试是非常困难的事情，因为js代码总是需要操作html元素。因此，这部分代码必须要在浏览器环境下才能运行。一旦脱离浏览器环境，代码便没办法执行。

可以利用nodejs进行单元测试，能够对前台有html操作的js也是可以进行单元测试的。

### 集成测试工具

模拟用户的输入，实现项目的整合测试。