---
title: 爬虫系统的核心模块
tags: [ispider]
---

1）随机IP代理器

使用IPProxyRepository.txt文档配置可使用的代理ip，冒号分隔ip和端口号。

启动应用时，自动加载到内存中，下面的两个数据结构中存储值ip代理数据

```
//IP地址代理库Map
private static Map<String, Integer> IPProxyRepository = new HashMap<>();
// keysArray是为了方便生成随机的代理对象
private static String[] keysArray = null;   
```

注：查看HttpUtil类，可以找到相应的实现代码

通过HttpUtil的getHttpContent方法，传递相关的url，获取网页数据。该方法使用HttpClient进行网页内容的获取，并返回网页字符串数据。同时设置了网络的超时，http头，ip代理等信息。很多代码都是硬编码实现的。

2）网页下载

IDownload接口定义了download方法，传递url，返回page对象，page对象是一个pojo类型的对象。主要设置了商品相关的字段信息，并使用content来保存网页下在内容。

HttpGetDownloadImpl实现类，内部使用HttpUtil来获取网页内容。

注：爬虫真正运行时，就是使用该实现类，通过传递url获取网页内容。然后进一步解析数据，保存到数据库中。

注2：这里page对象只是商品实体，如果需要其他实体对象，需要进一步扩展相关的pojo类。

3）网页解析器

这一部分使用开闭原则，通过自己实现IParser接口，实现不同页面的解析规则。

IParser接口，定义了parser方法，接收page对象，通过page对象的content可获取网页内容，然后解析数据到相应的属性中，从而完成数据的解析。

JDHtmlParserImpl和SNHtmlParserImpl两个实现类，用于分别解析京东网页和苏宁网页的实现类。内部使用HtmlCleanerr实现html文档的解析。对于网页不同部分使用不同的方式进行解析，如商品基础信息（使用xpath等语法获取），商品规格和商品评论都是json数据（使用jsonlib包进行解析下，如org.json.JSONObject类）。

注：HtmlCleaner是一个开源的Java语言的Html文档解析器

4）数据存储

mysql数据库，数据表字段与page中对于的商品实体类相对应。IStore为相应的接口，提供了store方法，通过接收page对象，将数据保存到数据库中。

MySQLStoreImpl实现类，使用commons-dbutils的jar包来保存数据（apache开源项目）。

5）运行过程