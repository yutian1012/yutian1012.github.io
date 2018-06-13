---
title: github开源项目云收藏（脚手架）
tags: [project]
---

1）利用Spring Initializr生成项目框架

访问地址：https://start.spring.io/

配置maven项目的GAV坐标值，选择spring boot版本为2.0.2，选择项目依赖为web和thymeleaf。

```
<groupId>com.favorites</groupId>
<artifactId>favorites-web-demo</artifactId>
<version>0.0.1-SNAPSHOT</version>
```

![](/images/project/favorites-web/web/spring-initializr.png)

下载压缩包，并导入到eclipse。

注：在导入前可先运行mvn命令下载相关的依赖，eclipse下载有些慢

```
# 定位项目目录，下载依赖
mvn clean compile
```

注：resources目录下的static和templates分别用来存放网站的静态资源（css，js等）和网页模板（thymeleaf编写的html文件）。

2）配置thymeleaf

编辑application.properties文件

```
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
# 暂时不开启页面缓存，开发过程中不建议开启
#spring.thymeleaf.cache=true
```

3）开启热部署

在pom中引入相关依赖devtools，该jar会监控项目中的文件是否发生变化，并重启项目

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    <scope>true</scope>
</dependency>
```

可以在application.properties文件中配置排除监控文件

```
# 排除static目录下的文件监控
spring.devtools.restart.exclude=static/**
```

4）编写一个测试类并启动web项目

编写Controller类：HelloDemoController

```
package com.favorites.favoriteswebdemo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloDemoController {
    
    @RequestMapping("/")
    public String index() {
        return "index";
    }
}
```

编写页面index.html

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>首页</title>
</head>
<body>
    hello favorite demo
</body>
</html>
```

访问地址：

```
localhost:8080
```

修改web端口，修改application.properties文件

```
server.port=9090
```