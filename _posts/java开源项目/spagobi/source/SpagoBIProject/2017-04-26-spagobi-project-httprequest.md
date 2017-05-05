---
title: 开源spagobi--http请求封装
tags: [spagobi]
---

针对用户请求信息，一般会将其存放到SessionContainer，RequestContainer和ResponseContainer中。

以登录功能为例查看信息的封装

```
http://localhost:8080/SpagoBI/servlet/AdapterHTTP?PAGE=LoginPage&NEW_SESSION=TRUE

//请求主体
isInternalSecurity=true&userID=biadmin&password=biadmin
```

### 1. SessionContainer类

该类继承了BaseContainer类，包含变量

_permanentContainer（放置持久信息的容器），同时为SessionContainer实例

_sharedContainer（放置共享信息的容器），同时为SessionContainer实例

```
SessionContainer sessionContainer = new SessionContainer(true);
```

### 2. RequestContainer类

成员变量

```
private static final long serialVersionUID = 1L;
private String _channelType = null;
private SourceBean _serviceRequest = null;
//管理requestContainer对应的sessionContainer
private SessionContainer _sessionContainer = null;
//线程局部变量中存放RequestContainer类型值
private static transient ThreadLocal _requestContainer = new ThreadLocal();
//线程局部变量中存放HttpServletRequest类型值
private static transient ThreadLocal _internalRequest = new ThreadLocal();
//线程局部变量中存放HttpServletResponse类型值
private static transient ThreadLocal _internalResponse = new ThreadLocal();
//线程局部变量中存放ServletConfig类型值
private static transient ThreadLocal _adapterConfig = new ThreadLocal();
```

1）在ProfileFilter类中的使用

dofilter方法中创建RequestContainer对象和SessionContainer对象

```
RequestContainer requestContainer = (RequestContainer) session.getAttribute(Constants.REQUEST_CONTAINER);
if (requestContainer == null) {
    // RequestContainer does not exists yet (maybe it is the
    // first call to Spago)
    // initializing Spago objects (user profile object must
    // be put into PermanentContainer)
    requestContainer = new RequestContainer();
    SessionContainer sessionContainer = new SessionContainer(true);
    requestContainer.setSessionContainer(sessionContainer);
    //提供了可以从HttpSession中获取requestContainer对象
    session.setAttribute(Constants.REQUEST_CONTAINER, requestContainer);
}

```

2）在AdapterHTTP类中的使用

service方法中创建RequestContainer

```
RequestContainer requestContainer = new RequestContainer();
RequestContainer.setRequestContainer(requestContainer);
...
//创建serviceRequest，并设置到RequestContainer中
SourceBean serviceRequest = new SourceBean(Constants.SERVICE_REQUEST);
requestContainer.setServiceRequest(serviceRequest);
//httpRequest对象的信息设置到requestContainer变量的_visibleAttributes集合中
setHttpRequestData(request, requestContainer);
...
String channelType = Constants.HTTP_CHANNEL;//值为HTTP
requestContainer.setChannelType(channelType);
//设置当前的HttpServletRequest到_internalRequest线程局部变量中
requestContainer.setInternalRequest(request);
//设置当前的HttpServletResponse到_internalResponse线程局部变量中
requestContainer.setInternalResponse(response);
//设置当前ServletConfig到_adapterConfig线程局部变量中
requestContainer.setAdapterConfig(getServletConfig());
```

注：setRequestContainer方法将requestContainer对象设置到线程局部变量_requestContainer中保存。

### 3. ResponseContainer类

AdapterHTTP类的service方法

```
ResponseContainer loopbackResponseContainer = ResponseContainer.getResponseContainer();
```

注：getResponseContainer方法是一个静态方法，从线程局部变量_responseContainer中获取ResponseContainer对象。



成员变量

```
private static final long serialVersionUID = 1L;
private String _businessType = null;
private String _businessName = null;
private String _publisherName = null;
private SourceBean _serviceResponse = null;
private SourceBean _loopbackServiceRequest = null;
private EMFErrorHandler _errorHandler = null;
private static transient ThreadLocal _responseContainer = new ThreadLocal();
```



### 4. BaseContainer类

该类内部包含变量：

_visibleAttributes（HashMap对象），放置设置的属性的key和value值

_hiddenAttributes（ArrayList对象），放置del删除的key

_patent（BaseContainer对象），作为父容器，提供信息的逐层获取

getAttribute方法，会先从_hiddenAttributes中找，如果存在则返回null（理解为删除了的属性），再到_visibleAttributes中查找，最后到_patent中查找并返回。

delAttribute方法，将key放入_hiddenAttributes集合中，并从_visibleAttributes中删除。