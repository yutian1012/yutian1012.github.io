---
title: spring boot thymeleaf视图模板
tags: [spring]
---

1）介绍

Thymeleaf是一款用于渲染XML，XHTML，HTML5内容的模板引擎，类似JSP，Velocity，FreeMaker等。它也可以轻易的与Spring MVC等Web框架进行集成作为Web应用的模板引擎。

Thymeleaf最大的特点是能够直接在浏览器中打开并正确显示模板页面，而不需要启动整个Web应用。Thymeleaf的模板语法并不会破坏文档的结构，模板依旧是有效的XML文档。

注：由于Thymeleaf使用了XML DOM解析器，因此它并不适合于处理大规模的XML文件

2）页面即原型

a.传统前端开发设计人员头疼问题

在Web开发过程中一个绕不开的话题就是前端工程师与后端工程师的写作，在传统Java Web开发过程中，前端工程师和后端工程师一样，也需要安装一套完整的开发环境，然后各类Java IDE中修改模板、静态资源文件，启动，重启，重新加载应用服务器，刷新页面查看最终效果。

实际上前端工程师的职责更多应该关注于页面本身而非后端，使用JSP，Velocity等传统的Java模板引擎很难做到这一点，因为它们必须在应用服务器中渲染完成后才能在浏览器中看到结果。

b.Thymeleaf所见即所得

Thymeleaf通过属性进行模板渲染不会引入任何新的浏览器不能识别的标签。整个页面直接作为HTML文件用浏览器打开，几乎就可以看到最终的效果，这大大解放了前端工程师的生产力，它们的最终交付物就是纯的HTML/CSS/JavaScript文件。

3）添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

注：之所以没有添加版本号，是因为pom中的parent元素指定了version版本。

4）添加thymeleaf模板的配置信息

在application.properties中添加配置

```
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

注：springboot就可以到/src/java/resources/templates/目录下查找模板

5）创建controller类

```
package com.example.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloThymeleafController {
    @RequestMapping("/helloThymeleaf")
    public String index() {
        return "hello";//返回视图名
    }
}
```

6）创建hello.html文件

在/src/java/resources/templates/目录下创建hello.html文件

```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>testThymeleaf</h1>
</body>
</html>
```

注：要在html中引入标签的命名空间

7）测试

```
mvn spring-boot:run
```

访问：http://localhost:8080/helloThymeleaf

8）缓存问题

Spring Boot支持的一些库中会使用缓存来提高性能。这里的thymeleaf模版引擎将缓存编译后的模板，以避免重复解析模板文件。

可通过设置相应的属性禁用掉缓存，编辑application.properties文件

```
#开发时关闭缓存,不然没法看到实时页面
spring.thymeleaf.cache=false
```