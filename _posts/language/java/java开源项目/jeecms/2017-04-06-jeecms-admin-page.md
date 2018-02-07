---
title: 开源的门户网站jeecms--后台页面
tags: [jeecms]
---

jeecms后台页面挂接的层次分析

### 1. 后台登陆成功

后台的首页：

1）在后台登录时通过CmsAuthenticationFilter类校验用户是否有权限登录

配置文件位于：shiro-context.xml文件

```
<bean id="authcFilter" 
class="com.jeecms.core.security.CmsAuthenticationFilter" parent="adminUrlBean">
    <property name="adminIndex" value="/jeeadmin/jeecms/index.do"/>
</bean>
```

注：认证通过则转到/jeeadmin/jeecms/index.do。

2）执行WelcomeAct类的index方法

```
@RequiresPermissions("index")
@RequestMapping("/index.do")
public String index(HttpServletRequest request) {
    return "index";
}
```

注：接收处理/jeeadmin/jeecms/index.do请求

使用/WebRoot/WEB-INF/jeecms_sys/index.html页面进行组织数据，返回客户端。

3）首页加载模块（top.do和main.do两个模块）

```
<frameset rows="71,*" frameborder="0" border="0" framespacing="0">
    <frame src="top.do" name="topFrame" noresize="noresize" id="leftFrame" />
    <frame src="main.do" name="mainFrame" id="mainFrame" />
</frameset>
```

4）WelcomeAct类top方法获取页头信息

top页面：/WebRoot/WEB-INF/jeecms_sys/top.html

该页面定义了后台管理的头信息

![](/images/open/jeecms/jeecms-admin-head.png)

main正文页面：/WebRoot/WEB-INF/jeecms_sys/main.html

```
<frameset cols="210,*" frameborder="0" border="0" framespacing="0">
    <frame src="left.do" name="leftFrame" noresize="noresize" id="leftFrame" />
    <frame src="right.do" name="rightFrame" id="rightFrame" />
</frameset>
```

5）如何添加新的功能到后台连接中

（1）添加新的top栏目

![](/images/open/jeecms/jeecms-head-add.png)

如果需要将功能添加到top菜单中，需要/WebRoot/WEB-INF/jeecms_sys/top.html文件中添加相应的连接。

（2）在左侧栏目中添加

左侧栏目稍微有些不同

一种类型是左侧树，是从数据库中获取树形目录（需要添加相应的记录，如内容栏目）。

![](/images/open/jeecms/jeecms-left-tree-add.png)

另外一种是左侧菜单，这种方式可以在相应的left.html加入相应的连接即可。

![](/images/open/jeecms/jeecms-left-add.png)

6）后台的资源

管理端（后台）用到的资源，比如js, css和img，放置在目录WebRoot/res下。

管理端（后台）用到的模板，放置在目录WebRoot/WEB-INF/jeecms_sys下

引用资源

```
<script src="${base}/thirdparty/My97DatePicker/WdatePicker.js" type="text/javascript"></script>
<script src="${base}/res/common/js/jquery.js" type="text/javascript"></script>
```

注：base的值是何时设置的

```
查看：RichFreeMarkerView类，在该类中初始化了base变量的值
```

### 2. 前端页面展示

1）直接访问http://localhost:8080/jeecms/

DynamicPageAct类的index方法会匹配前端的所有请求

```
@RequestMapping(value = "/", method = RequestMethod.GET)
    public String index(HttpServletRequest request,HttpServletResponse response, ModelMap model) {
```

返回页面：/WEB-INF/t/cms/www/default/index/index.html

注：CmsSite对象的getTplIndex返回值，数据库中js_site表的tpl_index字段值。

2）首页的显示

样式文件和js文件的引入

```
<link href="/${res}/css/html5.css" rel="stylesheet" type="text/css"/>
```

注：res变量是在CmsSite对象中获取的，可以在后台通过配置站点来设置。

引入文件：

```
[#include "../include/header.html"/]
[#include "../include/search_csi.html"/]
[#include "../include/navi.html"/] 导航栏页面
```

注：Freemarker默认是使用尖括号方式来包含标签，但是这样的写法有一个视觉问题，容易和html标签混合了。不友好，所以建议使用[] 来包含标签。

设置中括号代替尖括号的属性tag_syntax=auto_detect

```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="freemarkerSettings">
        <props>
            <prop key="tag_syntax">auto_detect</prop>
            ...
        </props>
    </property>
</bean>
```

配置方式：

```
可在spring配置文件中中配置相应的命令语法，也可使用代码配置
//设置尖括号语法和方括号语法,默认是自动检测语法  
//  自动 AUTO_DETECT_TAG_SYNTAX  
//  尖括号 ANGLE_BRACKET_TAG_SYNTAX  
//  方括号 SQUARE_BRACKET_TAG_SYNTAX  
cfg.setTagSyntax(Configuration.AUTO_DETECT_TAG_SYNTAX); 
```

注：默认两者都支持

3）前端的资源

用到的资源，如js, css和img，放置在目录WWebRoot/r下。

用到的模板，放置在目录WebRoot/WEB-INF/t下

引用资源

```
<script src="${resSys}/jquery.js" type="text/javascript"></script>
<img src="/${res}/images/top-ad.jpg">
```

注：resSys和res的值

```
resSys=/jeecms/r/cms
res=jeecms/r/cms/www/default
//查看FrontUtils.frontData方法
```

4）导航栏

导航生成配置文件：WebRoot\WEB-INF\t\cms\www\default\include\nav.html

注：如果前端需要定制导航信息，可以修改该文件，也可以通过在后台栏目中新建一个栏目，然后通过该文件的标签显示出导航信息（建议第二种方式）。

### 3. 动态页面的生成

动态页面主要涉及到后台的栏目、内容、模板、资源、配置等多个模块的配置。

1）首页

首页：/WEB-INF/t/cms/www/default/index/index.html

2）导航链接

（1）导航生成：使用freemarker标签cms_channel_list实现

```
[@cms_channel_list]
```

（2）标签定义，在jeecms-context.xml文件中配置

```
<bean id="cms_channel_list" class="com.jeecms.cms.action.directive.ChannelListDirective"/>
```

（3）标签配置，在jeecms-servlet-front.xml文件中配置freemarkerConfig的属性值

```
<property name="freemarkerVariables" value="#{propertyUtils.getBeanMap('directive.')}"/>
```

注：查找属性文件中开头为directive的配置属性。

在jeecms.properties中配置了相应的指令属性对应的的spring管理bean名称。

PropertyUtils.getBeanMap方法返回一个map对象作为freemarkerVariables属性的值。

```
public Map<String, Object> getBeanMap(String prefix) {
    //获取属性文件jeecms中的命令集合
    Map<String, String> keyMap = getMap(prefix);
    if (keyMap.isEmpty()) {
        return Collections.emptyMap();
    }
    Map<String, Object> resultMap = new HashMap<String, Object>(keyMap.size());
    String key, value;
    for (Map.Entry<String, String> entry : keyMap.entrySet()) {
        key = entry.getKey();
        value = entry.getValue();
        //将对象绑定到freemarkerConfiger的配置上
        resultMap.put(key, beanFactory.getBean(value, Object.class));
    }
    return resultMap;
}
```

注：PropertyUtils类实现了BeanFactoryAware，会自动将调用setBeanFactory，注入BeanFactory对象。

（4）导航路径的组成：

```
如新闻路径连接为：http://localhost:8080/jeecms/news/index.jhtml
主要有三部分组成：basePath,访问路径（news，栏目中设置的），index（栏目中是否包含内容决定的），后缀（.jhtml，配置--站点设置--动态页后缀）。
```

（5）动态路径生成（Channel类的getUrlDynamic方法）

```
public String getUrlDynamic(Boolean whole) {
    if (!StringUtils.isBlank(getLink())) {
        return getLink();
    }
    CmsSite site = getSite();
    StringBuilder url = site.getUrlBuffer(true, whole, false);
    url.append(SPT).append(getPath());//getPath即为栏目中设置的访问路径
    if (getHasContent()) {
        url.append(SPT).append(INDEX);//index
    }
    url.append(site.getDynamicSuffix());
    return url.toString();
}

//返回url路径http://localhost:8080/jeecms/news/index.jhtml
```

3）动态页的生成（以新闻栏目为例）

如新闻页面的生成：http://localhost:8080/jeecms/news/index.jhtml

（1）处理方法

当点击该连接，后台是如何处理的？DynamicPageAct.java类的dynamic方法，会匹配上面的连接，进行动态页面的生成。

```
@RequestMapping(value = "/**/*.*", method = RequestMethod.GET)
public String dynamic(HttpServletRequest request,
        HttpServletResponse response, ModelMap model) {
    // 尽量不要携带太多参数，多使用标签获取数据。
    // 目前已知的需要携带翻页信息。
    // 获得页号和翻页信息吧。
    int pageNo = URLHelper.getPageNo(request);
    String[] params = URLHelper.getParams(request);
    PageInfo info = URLHelper.getPageInfo(request);
    String[] paths = URLHelper.getPaths(request);
        
    int len = paths.length;
    if (len == 1) {
        // 单页
        return channel(paths[0],true, pageNo, params, info, request, response,
                    model);
    } else if (len == 2) {
        if (paths[1].equals(INDEX)) {
            // 栏目页
            return channel(paths[0],false, pageNo, params, info, request,
                        response, model);
        } else {
            // 内容页
            try {
                Integer id = Integer.parseInt(paths[1]);
                return content(id, pageNo, params, info, request, response,
                            model);
            } catch (NumberFormatException e) {
                log.debug("Content id must String: {}", paths[1]);
                return FrontUtils.pageNotFound(request, response, model);
            }
        }
    } else {
        log.debug("Illegal path length: {}, paths: {}", len, paths);
        return FrontUtils.pageNotFound(request, response, model);
    }
}
```

解决的问题是，找到使用的ftl来加载显示生成页面内容。路径长度为2且值为index表示是栏目页，调用channel方法，完成ftl的加载和解析。

（2）模板位置：

channel.getTplChannelOrDef()，即在栏目中设置的栏目模板位置。

```
public String getTplChannelOrDef() {
    String tpl = getTplChannel();//栏目中栏目模板的设置路径
    if (!StringUtils.isBlank(tpl)) {
        return tpl;
    } else {
        String sol = getSite().getSolutionPath();
        return getModel().getTplChannel(sol, true);
    }
}
```

4）动态内容的生成（以新闻内容页为例）

如查看新闻的内容：http://localhost:8080/jeecms/news/130.jhtml

（1）页面内容连接地址的生成

问题1：这里的130.jhtml是如何产生的？

查看新闻页面的ftl模板文件：WebRoot\WEB-INF\t\cms\www\default\channel\news.html

```
//新闻列表展示
[@cms_content_list  count='3' orderBy='4' channelId='75' channelOption='0' dateFormat='yyyy-MM-dd']
    [#list tag_list as a]
        <div class="item">
          <h1><a href="${a.url}" target="_blank">[@text_cut s=a.title len=17 append='...'/]</a></h1>
          <div class="des">[@text_cut s=a.description! len=72 append='...'/]</div>
          <span>${a.ctgName}<em>${a.date?string(dateFormat)}</em></span>
        </div>
    [/#list]
[/@cms_content_list]
```

查看标签定义，jeecms-context.xml文件中

```
<bean id="cms_content_list" class="com.jeecms.cms.action.directive.ContentListDirective"/>
```

内容list的获取：ContentListDirective父类的AbstractContentDirective的getData方法

连接地址的生成，查看Content类的getUrl方法

```
public String getUrl() {
    if (!StringUtils.isBlank(getLink())) {
        return getLink();
    }
    if (getStaticContent()) {
        return getUrlStatic(false, 1);
    } else if(!StringUtils.isBlank(getSite().getDomainAlias())){
        return getUrlDynamic(null);
    }else{
        return getUrlDynamic(true);
    }
}

public String getUrlDynamic(Boolean whole) {
    if (!StringUtils.isBlank(getLink())) {
        return getLink();
    }
    CmsSite site = getSite();
    StringBuilder url = site.getUrlBuffer(true, whole, false);
    url.append(SPT).append(getChannel().getPath());//栏目中设置的path访问路径
    //getId内容主键
    url.append(SPT).append(getId()).append(site.getDynamicSuffix());
    return url.toString();
}
```

注：这里的130即为新闻内容的主键（表jc_content的content_id键值）

（2）内容模板的定位

问题2：新闻内容模板页的渲染？即点击新闻链接所展示页面的生成

处理依然交由DynamicPageAct.java类的dynamic方法来实现。该方法内部会调用content方法。

模板定位：content.getTplContentOrDef(content.getModel())

```
//Content类
public String getTplContentOrDef(CmsModel model) {
    String tpl = getTplContent();
    if (!StringUtils.isBlank(tpl)) {
         return tpl;
    } else {
        return getChannel().getTplContentOrDef(model);
    }
}
```

调用channel类的getTplContentOrDef方法

```
//channel类
public String getTplContentOrDef(CmsModel contentModel) {
    //查询channelModel获取设置的内容模板路径
    String tpl = getModelTpl(contentModel);
    if (!StringUtils.isBlank(tpl)) {
        return tpl;
    } else {
        String sol = getSite().getSolutionPath();
        return contentModel.getTplContent(sol, true);//添加内容时设置的模板
    }
}
```

注：最终获取的信息是来自jc_channel_model表中的值。

页面中的保存值WebRoot\WEB-INF\jeecms_sys\channel\edit.html文件中

```
<div id="modelsContainer">
<#list channel.channelModels as cm>
<input name='modelIds' type='hidden' value="${cm.model.id}"/>
<input name='tpls' type='hidden' value="${cm.tplContent!}" />
<input name='mtpls' type='hidden' value="${cm.tplMoibleContent!}" />
</#list>
</div>
//tpls会设置到jc_channel_model表的tpl_content字段上
```

内容模板的设置：（添加修改内容时可以指定模板位置）

![](/images/open/jeecms/jeecms-content-template.png)
