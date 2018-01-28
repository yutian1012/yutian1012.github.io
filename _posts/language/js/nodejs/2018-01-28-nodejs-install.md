---
title: nodejs安装
tags: [nodejs]
---

### windows安装nodejs

1）下载软件

下载地址：https://nodejs.org/en/

安装过程相对简单，只需要根据提示下一步的执行就可以成功安装。

2）环境变量

检测PATH环境变量是否配置了Node.js，在cmd输入命令

```
echo %path%
```

3）查看安装

```
node --version
或
node -v
```

4）运行js文件

新建一个helloworld.js文件

```
console.log("Hello World");
```

运行

```
node helloworld.js
```

5）运行命令行交互

```
# 使用node进入交互功能
node

console.log("hellowlorld");

# ctrl+c输入两次，退出交互界面
```