---
title: nodejs包管理器
tags: [nodejs]
---

NPM全称（Node Package Manager），即node的包管理器。

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题。可以方便的安装、更新和卸载第三方包。这里的包可以理解为基于nodejs的第三方库或插件。在这些插件和库的基础上能够开发出更强大的功能。

### 安装npm

1）查看是否安装了npm

由于新版的nodejs已经集成了npm，所以之前npm也一并安装好了。

```
npm -v
```

2）如果单独安装需要使用源码安装

参考：https://www.cnblogs.com/seanlv/archive/2011/11/22/2258716.html

首先，下载源码（github），使用node进行安装。官方也提供了安装包msi，同时使用make-install也能够安装npm。

3）理解全局作用域

在npm install插件时，如何指定-g，表示将插件安装到全局目录中。

```
# 查看npm安装插件的全局目录
npm root -g
```

### 主要作用

方便安装，更新，删除node插件、第三方库。

1）查看npm版本信息

```
npm -v
```

2）npm查看软件版本

```
# 查看grunt软件远程支持的版本号，然后可以下载对应的软件版本
npm view grunt versions --json
# 查看本地安装的软件版本，如果当前目录安装了该插件
npm ls grunt
```

3）安装软件

```
# 在当前目录下下载grunt插件（代码混淆工具）
npm install grunt
# 安装指定版本，@符号后跟版本号
npm install grunt@1.0.0
```

4）移除软件

```
# 从当前目录中移除grunt插件
npm remove grunt
```

5）安装更新

```
# 更新grunt插件到库中支持的最新版本
npm update grunt
```

注：如果不想更新到最新版本，只能remove掉，然后再install指定的版本。

6）将安装的软件信息写入package.json文件中（自定义的npm软件）

如果一个模块不在package.json文件之中，可以单独安装这个模块，并使用相应的参数，将其写入package.json文件之中。

```
# --save参数表示将该模块写入dependencies属性
npm install express --save
# --save-dev表示将该模块写入devDependencies属性
npm install express --save-dev

```

注：在自行创建的项目中如果要引入第三方的依赖，可以在安装依赖时同时将依赖写入到package.json包中。