---
title: github开源项目云收藏（thymeleaf模板布局）
tags: [project]
---

页面布局的要点

1）定义模板显示片段fragment

fragment片段定义语法：如th:fragment="header"，这样就定义了一个名为header的fragment，具体查询下面代码。

```
<!-- header.html页面 -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<!-- body标签定义header页面的内容 -->
<body>
    <!-- 
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

2）引用模板片段fragment

使用th:include和th:replace命令进行模板片段的引用。

a.使用replace方式将模板中的内容替换到布局页面 

页面模板的引用方式，templatename::fragmentname（即模板名::片段名），如下面引用header.html页面定义的header片段fragment（th:replace="fragments/header :: header"）

```
<!-- layout.html页面 -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <!-- 定义htmlhead片段代码 -->
  <head th:fragment="htmlhead">
    <meta charset="utf-8"></meta>
   
    <!-- 添加样式文件 link ... -->
    
  </head>

  <body class="layout-fixed">
    <!-- 
        定义layout片段
        -->
    <div th:fragment="layout"  class="wrapper" >
        <div id="headerfragment" th:replace="fragments/header :: header"> 
            Header
        </div>
        ...
    </div>
    
    <!-- 下方放置待引入的script脚本文件 -->
</body>
</html>
```

注：这里我们使用的是div标签，并且定义了id属性，以及在标签体内定义了相关的文本。使用replace命令后，这里会完全被替换成模板片段标签，即这里的div被替换成了header标签，div的属性和内容也同时被替换掉了。

![](/images/project/favorites-web/web/layout-replace.png)

b.使用include方式将模板中的内容替换到布局页面

在上面的示例代码上将replace替换成include命令，div标签的属性和内容不变。使用include命令的效果，只将定义的模板内容替换了div标签内容，div标签和标签的属性不发生变化。

```
<!-- layout.html页面 -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <!-- 定义htmlhead片段代码 -->
  <head th:fragment="htmlhead">
    <meta charset="utf-8"></meta>
   
    <!-- 添加样式文件 link ... -->
    
  </head>

  <body class="layout-fixed">
    <!-- 
        定义layout片段
        -->
    <div th:fragment="layout"  class="wrapper" >
        <div id="headerfragment" th:include="fragments/header :: header"> 
            Header
        </div>
        ...
    </div>
    
    <!-- 下方放置待引入的script脚本文件 -->
</body>
</html>
```

![](/images/project/favorites-web/web/layout-include.png)

注：include和replace的主要区别是是否替换掉外层标签及其属性。

3）定义标签的局部变量

使用命令th:with，可以定义标签体的局部变量。

下面结合include和with命令来动态的向模板片段中传递参数。

a.修改header.html页面内容（截取修改片段）

```
<header th:fragment="header">
    <nav>
        <div th:text="${nav}"></div>
    </nav>
</header>
```

注：导航标签体的div显示内容不在使用固定的字符串，而是使用${nav}来引用变量

b.使用with命令定义nav局部变量

修改layout.html页面，引用header.html，并定义nav局部变量，设置变量的值传递到模板中

```
<div th:fragment="layout"  class="wrapper" >
    <div id="headerfragment" th:include="fragments/header :: header" 
        th:with="nav='favorites'"> 
        Header
    </div>
    ...
</div>
```

查看运行结果

![](/images/project/favorites-web/web/layout-with.png)

4）使用data-layout-fragment定义待替换的内容区块

在一个网页中，一般头部导航条，左右两侧的导航和工具，以及页面下方的footer版权信息都是固定的。而只有中间内容展示区会是变化的，不同页面内容区域不同。如何实现不同页面替换掉布局页面的内容展示区是关键。

a.修改layout.html页面，定义布局

```
<div th:fragment="layout"  class="wrapper" >
    <div th:replace="fragments/header :: header" >Header</div>
    <div th:replace="fragments/left :: left">left</div>
    <div th:replace="fragments/sidebar :: sidebar">sidebar</div>

    <div id="content" data-layout-fragment="content" >
        替换文档内容
    </div>

    <div th:replace="fragments/footer :: footer">footer</div>
</div>
```

注：上面的代码中定义了网页的整体布局，包括header都，左右工具条，中间内容展示区域，以及页面底部的版权信息。其中内容展示区是动态的，不同页面要替换该区域展示属于自己页面的内容。在layout.html中使用data-layout-fragment命令来定义这部分区块。

注2：一般布局页面不仅仅定义页面的整体框架，同时将页面的css样式也一并设置好。

b.定义home.html，展示用户的首页

该页面要替换掉data-layout-fragment区域的内容，显示用户的信息

```
<html xmlns:th="http://www.thymeleaf.org" data-layout-decorate="fragments/layout">

  <body>
      <!-- data-layout-fragment（相当于layout:fragment） -->
      <section data-layout-fragment="content">
        home页面内容
      </section>
  </body>
  <script type='text/javascript'>
 </script>
</html>
```

注：data-layout-decorate命令指定使用的布局页面来装饰该home.html页面。data-layout-fragment指定替换掉布局页面的那部分区域（布局页面可能会定义多个待替换区域，这里需要指定需要替换的区域）。这里替换的内容只传递了固定的字符串，具体实际情况，会从后台服务中加载出用户的信息，并显示到该区域。

5）总结（使用到的命令）

a.th:fragment，定义模板片段（模板文件中可以定义不止一个片段fragment）

b.th:include和th:replace，引用模板片段（注意两者的区别）

c.th:with可以向模板片段中传递局部变量，动态展示模板信息

d.data-layout-fragment定义和指定需要动态替换的内容展示区域

e.data-layout-decorate用来指定页面需要使用哪个布局页面来装饰