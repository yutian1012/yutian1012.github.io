---
title: chrome浏览器安装离线插件
tags: [browser]
---

以安装Batarang插件为例，chrome实现离线插件。

1）插件介绍（js调试）

Batarang是AngularJS调试的Google-chrome插件，安装后可以在google浏览器中对AngularJS应用代码进行调试。

2）下载地址

下载：https://github.com/angular/angularjs-batarang/releases

下载压缩包后解压文件。

3）安装

参考：https://github.com/angular/batarang/

chrome浏览器右上角按钮--更多工具--扩展程序，勾选开发者模式。点击加载已解压的扩展程序按钮，选择下载的batarang解压文件。

![](/images/tools/browser/chrome/plugin.png)

4）使用该插件

在chrome浏览器中按F12，就能看到AngularJs标签页。

![](/images/tools/browser/chrome/batarang-plugin.png)

5）问题

使用的batarang-0.10.9版本，安装完成后，F12打开开发者工具，切换到angularJs标签页，显示空白。

参考：http://blog.csdn.net/u013236043/article/details/73957808

解决方法：

a.打开压缩包中提供的测试文件index.html

在解压包下存在一个example文件夹，使用chrome浏览器打开该文件夹下的index.html（测试文件）报错，angular is not defined。因此，控件的运行需要angularJs的支持，可以使用nodejs的npm包管理工具下载。

b.定位问题

打开index.html，发现文件中引入了angular文件

```
<script src="../node_modules/angular/angular.js"></script>
```

而压缩包中并没有提供node_modules文件夹，因而更不可能存在angularjs文件了。

c.使用npm管理工具下载angular

```
# 在软件的解压目录下创建node_modules文件夹
mkdir node_modules
# 切换进入该文件夹
cd node_moudules
# 运行npm下载angular，会在运行的目录下保存下载的软件包
npm install angular
```

d.再次打开index.html，能够正确的使用angular功能了。