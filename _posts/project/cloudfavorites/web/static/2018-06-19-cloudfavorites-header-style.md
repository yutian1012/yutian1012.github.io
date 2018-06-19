---
title: github开源项目云收藏（header样式）
tags: [project]
---

header.html页面样式实现

1）header.html内容

查看该页面可以清楚的看到导航条的组织结构，导航条目都包在nav标签中。使用div包裹logo图标，同时使用平级的div包裹相关的操作图标，ul和li分别用来定义菜单和子菜单。

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <!-- h5标签 header 定义header导航头，header可理解为有语义的div -->
    <header class="topnavbar-wrapper" th:fragment="header">
      <!-- h5标签 nav 定义导航项目
      <nav class="navbar topnavbar" role="navigation">
         <div class="navbar-header">
            <a class="navbar-brand" href="/index">
                <!-- 定义logo图标 -->
                <div class="brand-logo">
                  <img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}"/>
                </div>
                <!-- 定义折叠显示的logo图标 -->
                <div class="brand-logo-collapsed">
                  <img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}" />
                </div>
            </a>
         </div>

         <div class="nav-wrapper">
            <!-- 使用无序列表，折叠按钮，用来折叠显示左侧栏位 -->
            <ul class="nav navbar-nav">
               <li>
                  <a title="缩小/扩大侧栏" class="hidden-xs" data-toggle-state="aside-collapsed" href="#"> 
                    <em class="fa fa-navicon"></em>
                  </a>

                  <a title="缩小/扩大侧栏" class="visible-xs sidebar-toggle" data-no-persist="true" data-toggle-state="aside-toggled" href="#"> 
                    <em class="fa fa-navicon"></em>
                  </a>
               </li>
            </ul>
            
            <!-- 定义页面右上角处的操作图标 -->
            <ul class="nav navbar-nav navbar-right">
                <!-- 搜索图标 -->
               <li>
                  <a title="搜索" data-search-open="" href="#">
                     <em class="icon-magnifier"></em>
                  </a>
               </li>

               <li class="dropdown dropdown-list">
                  <a title="消息通知" data-toggle="dropdown" href="#">
                     <em class="icon-bell"></em>
                     <div class="label label-danger" id="noticeNum"></div>
                  </a>
                  <!-- 消息通知下的子菜单 -->
                  <ul class="dropdown-menu animated flipInX">
                     <li>
                        <div class="list-group">
                           <a class="list-group-item" href="javascript:void(0);" onclick="showNotice('at')">
                              <div class="media-box">
                                 <div class="pull-left">
                                    <em class="fa fa-at fa-2x text-info"></em>
                                 </div>
                                 <div class="media-box-body clearfix">
                                    <p class="m0">@我的</p>
                                   ...
                                 </div>
                               </div>
                            </a>
                            ....
                        </div>
                     </li>
                  </ul>
               </li>

               <li>
                  <a title="浏览记录" href="javascript:void(0);" onclick="locationUrl('/lookRecord/standard/my/0','lookRecord');">
                     <em class="icon-clock"></em>
                  </a>
               </li>
               ....
            </ul>

         </div>
         <!-- 搜索提交表单 -->
         <form class="navbar-form"  role="search">
            <div class="form-group has-feedback">
               <input id="searchKey" type="text" class="form-control" placeholder="输入并且按回车确定..." />
               <div class="fa fa-times form-control-feedback" data-search-dismiss=""></div>
            </div>
            <button  class="hidden btn btn-default" type="button" >提交</button>
         </form>
      </nav>
     <script type='text/javascript'>
        document.onkeydown = function(e){ 
            if(!e) e = window.event;//火狐中是 window.event 
            if((e.keyCode || e.which) == 13){
                window.event ? window.event.returnValue = false : e.preventDefault();
                var key=document.getElementById("searchKey").value;
                 if(key!=''){
                    locationUrl('/search/'+key,"");
                 }
            } 
        } 
    </script>
   </header>
</body>
</html>
```

2）不加样式前直接访问

在HelloDemoController类中添加header方法，用于定位展示header.html页面

```
/**
 * 直接访问布局页面
 * @return
 */
@RequestMapping("/header")
public String header() {
    return "fragments/header";
}
```

3）设置样式

a.引入图标样式库simple-line-icons

```
<link rel="stylesheet" th:href="@{/vendor/simple-line-icons/css/simple-line-icons.css}"/>
```

该css文件提供了相关的图标，只需要引入相关的class就可以加载对应的图标。

b.navbar导航条

引入bootstrap的css样式

```
<link rel="stylesheet" th:href="@{/media/css/bootstrap.css}" />
```

在bootstrap中的component查找navbar控件的使用

![](/images/project/favorites-web/style/bootstrap-navbar.png)

参考：https://getbootstrap.com/docs/4.0/components/navbar/

c.引入bootstrap图标库font-awesome

```
<link rel="stylesheet" th:href="@{/vendor/fontawesome/css/font-awesome.min.css}" />
```

d.引入animate.css样式

该样式用来实现动态的效果，通过绑定class属性来实现不同的动画效果。页面中子菜单的弹出时使用了animate样式。

```
<link rel="stylesheet" th:href="@{/vendor/animate.css/animate.min.css}"/>
```