---
title: 开源spagobi--AdapterHTTP代码分析（重点）
tags: [spagobi]
---

实现http请求的适配处理类。

### 1. AdapterHTTP类介绍

首先该类继承了HttpServelt，因此，可以接收用户的请求，并对其请求参数进行处理，调用service，最后设置路径跳转，完成页面渲染处理。

注：该类与spring mvc中的DispatchServlet相似，实现了请求映射到相应的方法。

### 2. 处理过程

封装处理请求信息，设置跳转路径，检查session等操作后。

获取CoordinatorIFace的实例对象，调用service方法，最后返回。

```
ResponseContainer,RequestContainer,HttpSession等对象
...
coordinator = DispatcherManager.getCoordinator(requestContext);
...
coordinator.service(serviceRequest, serviceResponse);
...
render(requestContext, serviceException);
```

### 3. DispatcherManager类

该类的转发或分发集合实例，是由registerDispatcher方法实现的。而该方法调用是在DispatcherInitializer中实现的。DispatcherInitializer类配置在了initializers.xml文件中。该类初始化的操作时在ConfigServlet容器启动时执行的，具体可查看InitializerManager类的init方法。

DispatcherInitializer会向DispatcherManager对象中注入ActionDispatcher和ModuleDispatcher两个类，查看WEB-INF/conf/config/dispatchers.xml文件。

1）dispatchers.xml文件内容：

```
<DISPATCHERS>
    <DISPATCHER class="it.eng.spago.dispatching.action.ActionDispatcher" />
    <DISPATCHER class="it.eng.spago.dispatching.module.ModuleDispatcher" />
</DISPATCHERS>
```

```
List distributions = configSingleton.getFilteredSourceBeanAttributeAsList("DISTRIBUTIONS.DISTRIBUTION","BUSINESS_NAME", businessName);//返回空
...
coordinator = dispatcher.getCoordinator(requestContext);
```

注：这里的dispatcher为ModuleDispatcher的实例。configSingleton.getFilteredSourceBeanAttributeAsList会检索配置文件中的DISTRIBUTIONS标签，

注2：在所有的xml中没有发现DISTRIBUTIONS标签，因此返回空。

2）查看ModuleDispatcher类的getCoordinator方法

```
public String getBusinessType(RequestContextIFace requestContext) {
    return "PAGE";
} 

public String getBusinessName(RequestContextIFace requestContext) {
    SourceBean serviceRequest = requestContext.getRequestContainer().getServiceRequest();
    String pageName = (String)serviceRequest.getAttribute(Constants.PAGE);
    return pageName.toUpperCase();
} 

public String getPublisherName(RequestContextIFace requestContext) {
    ConfigSingleton configure = ConfigSingleton.getInstance();
    SourceBean actionBean = (SourceBean)configure.getFilteredSourceBeanAttribute(Constants.PAGE_PATH, "NAME", getBusinessName(requestContext));//PAGES.PAGE
    if (actionBean == null) {
        return null;
    }
    String publisherName = (String)actionBean.getAttribute(Constants.PUBLISHER_NAME);
    return publisherName;
}

public CoordinatorIFace getCoordinator(RequestContextIFace requestContext) {
    return new ModuleCoordinator(getBusinessType(requestContext), getBusinessName(requestContext), getPublisherName(requestContext));
}
```

注：Constants.PAGE_PATH的值为"PAGES.PAGE"，会查找xml文件的PAGES标签并查找Name属性，属性值为businessName的数据。

### 4. 实例

1）请求地址：

```
http://localhost:8080/SpagoBI/servlet/AdapterHTTP?PAGE=LoginPage&NEW_SESSION=TRUE
```

注：ModuleDispatcher处理时，获取请求中的信息，BusinessType为PAGE，BusinessName为LoginPage，publisherName为null。

查看配置文件：

```
WEB-INF/conf/webapp/pages.xml
内容如下：
<PAGE name="LoginPage" scope="REQUEST" title="Document template build page" >
    <DEPENDENCIES>
        <DEPENDENCE source="LoginPage" target="LoginModule">
            <CONDITIONS/>
            <CONSEQUENCES/>
        </DEPENDENCE>
    </DEPENDENCIES>
    <MODULES>
        <MODULE keep_instance="false" keep_response="false" name="LoginModule"/>
    </MODULES>
</PAGE>
...
```

另外：主master.xml文件

```
<!-- webapp --> 

<CONFIGURATOR path="/WEB-INF/conf/webapp/messages.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/modules.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/pages.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/presentation.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/publishers.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/authorizations.xml" /> 
...
```

注：前面的注释表示配置信息所属的模块，因此，这些配置属于webapp模块下。

2）coordinator.service方法调用

coordinator为ModuleCoordinator的实例，调用service时，内部会构造出PageIFace对象，再调用page的service方法

```
PageIFace page =PageFactory.getPage(getRequestContainer(), getBusinessName());
...
page.service(serviceRequest, serviceResponse);
```

PageFactory.getPage方法

```
ConfigSingleton configure = ConfigSingleton.getInstance();
SourceBean pageBean =(SourceBean) configure.getFilteredSourceBeanAttribute(Constants.PAGE_PATH,"NAME",pageName);
...
String pageClass = (String) pageBean.getAttribute("CLASS");
if (pageClass == null) {
    pageClass = "it.eng.spago.dispatching.module.DefaultPage";
} 
...
page = getInstance(pageClass, pageBean);//利用类的反射获取对象
```

注：getInstance获取DefaultPage实例，并调用init方法使用PageBean进行初始化对象。

DefaultPage类的service方法

```
ArrayList moduleConfigs =(ArrayList)getConfig().getAttributeAsList("MODULES.MODULE");//PAGE标签下的MODULES标签配置
...
ModuleIFace module = ModuleFactory.getModule(moduleName);//LoginModule
...
module.service(...)//最终调用
```

注：getConfig方法获取的是PAGE对象的xml配置信息，

ModuleFactory类的getModule方法

```
//查找配置文件<CONFIGURATOR path="/WEB-INF/conf/webapp/modules.xml" />
SourceBean moduleBean = (SourceBean)configure.getFilteredSourceBeanAttribute("MODULES.MODULE", "NAME", moduleName);
...
//初始化配置
SourceBean moduleConfig = (SourceBean)moduleBean.getAttribute("CONFIG");
...
//生成对象
ModuleIFace module = null;
module = (ModuleIFace)Class.forName(moduleClass).newInstance();
...
//初始化
((InitializerIFace)module).init(moduleConfig);
```
