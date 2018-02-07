---
title: myeclipse安装groovy插件
tags: [groovy]
---

参考：https://github.com/groovy/groovy-eclipse/wiki

注：eclipse的groovy插件版本集合（支持不同版本的eclipse），需要针对不同版本的eclipse下载不同的groovy插件，否则可能出现错误。

### eclipse安装groovy插件

安装时需要针对不同版本的eclipse选择不同的插件。

1）安装插件

在http://marketplace.eclipse.org/中查找groovy插件并安装。

查看eclipse的版本号，help菜单中可以查看到当前安装的eclipse的版本

```
Release 4.3.0
```

下载的插件地址

```
groovy - http://dist.springsource.org/release/GRECLIPSE/e4.3/
```

2）新建groovy project项目(项目名groovyJava)

![](/images/groovy/info/newgroovyproject.png)

3）添加junit测试

![](/images/groovy/info/newgroovyprojectjunit.png)

4）新建Hellowold类并测试

```
class Helloworld {
    @Test
    void testHelloworld(){
        println "helloworld";
    }
}
```

![](/images/groovy/info/groovy-helloworld2.png)

### myeclipse安装groovy插件

查看确认myeclipse集成的Eclipse的版本号。

在myeclipse的安装目录下的readme文件夹下查看readme_eclipse.html

```
Release 4.3.0
```

4）确定出需要安装的groovy插件地址

```
groovy - http://dist.springsource.org/release/GRECLIPSE/e4.3/
```

重新安装插件，并测试。