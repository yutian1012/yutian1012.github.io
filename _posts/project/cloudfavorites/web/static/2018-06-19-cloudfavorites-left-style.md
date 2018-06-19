---
title: github开源项目云收藏（left样式）
tags: [project]
---

left.html页面样式实现

1）left.html内容

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
   <!-- h5元素 aside，定义侧边栏 -->
   <aside class="aside" th:fragment="left">
      <div class="aside-inner">
         <!-- 左侧导航 -->
         <nav class="sidebar" data-sidebar-anyclick-close="">
            <ul class="nav">
               <!-- 个人图片以及简介信息 -->
               <li class="has-user-block">
                  
               </li>

               <!-- 定义导航信息 -->
               <li class="nav-heading ">主导航</li>
               <li class="" id="home">
                  <a title="首页"  href="javascript:void(0);"  onclick="locationUrl('/standard/my/0','home')">
                     <em class="icon-home"></em>
                     <span>首页</span>
                  </a>
               </li>
               <li class=" " id="unread">
               </li>
               <li class=" " >
                  <a data-toggle="collapse" title="收藏夹" href="#favorites">
                     <em class="icon-folder"></em>
                     <span>收藏夹</span>
                  </a>
                  <ul class="nav sidebar-subnav collapse" id="favorites">
                     <li class="sidebar-subnav-header">收藏夹</li>
                     <li id="newFavortes" class=" ">
                        <a title="+ 新建收藏夹" href="javascript:void(0);" onclick="locationUrl('/newFavorites','newFavortes')">
                           <span>+ 新建收藏夹</span>
                        </a>
                     </li>
                  </ul>
               </li>
              ...
            </ul>
         </nav>
      </div>
   </aside>
</body>
</html>
```

2）直接页面中查看

在HelloDemoController类中添加left方法

```
/**
 * 直接访问布局页面
 * @return
 */
@RequestMapping("/left")
public String left() {
    return "fragments/left";
}
```