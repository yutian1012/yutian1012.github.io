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

### 主要作用

方便安装，更新，删除node插件、第三方库。