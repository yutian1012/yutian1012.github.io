---
title: github开源项目云收藏（thymeleaf模板布局）
tags: [project]
---

在相关的脚手架项目中已经引入了web和thymeleaf等相关模块的依赖，这里结合bootstrap制作前端页面显示。

利用thymeleaf布局+bootstrap前端框架搭建项目的前端。

1）引入依赖

```
<!-- thymeleaf layout布局依赖 -->
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

2）布局页面

页面一般包含五大部分

a.header（头信息，一般为图片logo等，navigation等）

b.left（左边栏目信息，一般是左侧树形结构）

c.content（中间内容显示区域）

d.sidebar（右侧工具栏区域）

f.footer（底部版权，电话等信息）

![](/images/project/favorites-web/web/layout.png)

3）创建简单布局实例

在resources/template目录下创建fragment目录，在该目录下创建header.html，footer.html，left.html，sidebar.html以及layout.html页面。

注：在定义模板内容时一般都放置在body标签体内。

注2：layout.html是布局页面，而其它页面是模板页面。系统中的页面都是基于布局页面进行扩展的。

a.新建header页面

网页头部元素，展示logo，导航条等部分

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<!-- body标签定义header页面的内容 -->
<body>
    <!-- fragment片段定义语法：如th:fragment="header"，这样就定义了一个名为header的fragment 
        这里header标签是h5的语义标签
    -->
    <header th:fragment="header">
        <nav>
            <!-- 设置导航栏信息 -->
            <div>导航信息测试</div>
        </nav>
    </header>
</body>
</html>
```

注：使用th:fragment定义页面显示片段内容，只需要在相应的页面中引入此片段即可

b.新建layout.html页面

在该布局页面中引入header页面，并查看效果

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <!-- 使用th:replace引入header定义的片段header
        引入fragment语法为：模板名 :: 片段名
     -->
    <div th:replace="fragments/header :: header">Header</div>
</body>
</html>
```

在HelloDemoController类中创建layout方法，用于跳转到layout.html页面

```
/**
 * 直接访问布局页面
 * @return
 */
@RequestMapping("/layout")
public String layout() {
    return "fragments/layout";
}
```

运行查看页面

![](/images/project/favorites-web/web/th-fragment-layout.png)

c.创建footer.html页面

该页面展示页脚信息，如版权等信息

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <!-- footer是h5标签 -->
    <footer th:fragment="footer">
        <span>&copy; 2016 - 2017</span>
     </footer>
</body>
</html>
```

d.创建left.html页面

该页面用于显示左侧树连接，对系统功能的导航

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <!-- aside是h5标签，用来定义左侧栏位 -->
    <aside class="aside" th:fragment="left">
        <div class="aside-inner">
            左侧树
        </div>
    </aside>
</body>
</html>
```

e.创建sidebar.html页面

该页面主要位于网页的右侧，定义常用的工具栏位

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body >
    <aside class="offsidebar" th:fragment="sidebar" >
        <div>
            右侧工具栏位
        </div>
    </aside>
</body>
</html>
```

f.完善layout.html页面

在layout.html页面中引入其他模板，并定义将被内容替换的区域

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <!-- 定义htmlhead片段代码 -->
  <head th:fragment="htmlhead">
    <meta charset="utf-8"></meta>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"></meta>
    <meta name="renderer" content="webkit" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"></meta>
    <meta name="description" content=""></meta>
    <meta name="author" content=""></meta>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"></meta>
    <link rel="icon" href="/img/icon.ico" type="image/x-icon" />
    <title >设置标题</title>
    
    <!-- 添加样式文件 link ... -->
    
  </head>

  <body class="layout-fixed">
    <!-- 
        定义layout片段
        使用replace方式将模板中的内容替换到布局页面 
        页面的引用信息 templatename::fragmentname，即模板名::片段名
        -->
    <div th:fragment="layout"  class="wrapper" >
        <div th:replace="fragments/header :: header">Header</div>
        <div th:replace="fragments/left :: left">left</div>
        <div th:replace="fragments/sidebar :: sidebar">sidebar</div>
        <!-- data-layout-fragment（相当于layout:fragment） -->
        <div id="content" data-layout-fragment="content" >
            替换文档内容
        </div>
        <div th:replace="fragments/footer :: footer">footer</div>
    </div>
    
    <!-- 下方放置待引入的script脚本文件 -->
</body>
</html>
```

注意：该页面是系统的整体结构页面，系统中的其他页面都是基于该页面展示的，只是替换了该页面的部分内容而已。

该页面定义了两个片段htmlhead和layout，同时使用th:replace命令将其他模板页面引入到布局页面中

注：该页面定义了head头显示信息，引入了系统的样式文件，同时在下方引入了script脚本，是一个完整的页面。

4）创建home.html页面

下面，我们创建用户的主页面，该页面基于layout布局页面，并替换掉不仅页面的内容显示区域。

```
<html xmlns:th="http://www.thymeleaf.org" data-layout-decorator="fragments/layout">
  
</html>
```

注：这里什么都不做，只是引入布局页面，使用data-layout-decorator来指定使用布局页面装饰home.html页面。

在HelloDemoController类中创建home方法，用来跳转到home.html页面

```
/**
 * 用户的主页
 * @return
 */
@RequestMapping("/home")
public String home() {
    return "home";
}
```

注：home.html定义在resources/templates目录下。

![](/images/project/favorites-web/web/data-layout-decorator.png)

5）编辑home.html页面

上面的代码只是使用布局页面进行了装饰，并没有设置替换功能

装饰时对layout中定义的htmlhead片段进行部分修改，这里通过传递局部变量定义修改。

```
<html xmlns:th="http://www.thymeleaf.org" data-layout-decorator="fragments/layout">
  <!-- 
    上面的data-layout-decorator(相当于layout:decorate)指定了使用fragments目录下layout.html页面作为布局页面
        
        th:include标签指定了layout模板中定义的fragment htmlhead片段来修饰head标签体
        注：th:include与th:replace稍有不同，include表示将片段内容替换到该标签体中，而replace是整个替换，包括标签本身也会被替换掉
        
        th:with用来定义局部变量，下面的定义信息中，可以使用${title}来访问定义的局部变量
   -->
  <head th:include="fragments/layout :: htmlhead"  th:with="title='favorites'"></head>
</html>
```

注：使用th:with定义局部变量，该变量作用域在head标签内。

注2：这里使用th:include而不是使用th:replace，需要注意两者的区别。

修改layout.html，在head标签中应用这里定义的title局部变量，在title标签上使用th:text来引用title局部变量

```
...
  <head th:fragment="htmlhead">
    ....
    <title th:text="${title}"></title>
    ....
  </head>
  ....
```

原程序中home.jsp登录后的首页，加载了collect/standard.html页面，如何实现的？

这里是通过js来调用的，具体查看layout.js