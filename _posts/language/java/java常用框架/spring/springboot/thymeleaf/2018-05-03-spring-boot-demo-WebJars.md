---
title: spring boot webJars统一管理js，css等资源文件
tags: [spring]
---

WebJars是一个很神奇的东西，可以让大家以jar包的形式来使用前端的各种框架、组件。

1）介绍

WebJars是将客户端（浏览器）资源（JavaScript，Css等）打成jar包文件，以对资源进行统一依赖管理。WebJars的jar包部署在Maven中央仓库上。在项目中只要将相应的依赖引入，即可使用。

2）解决前端资源的依赖处理

传统web开发对于JavaScript，Css等前端资源包，只能采用拷贝到webapp下的方式，这样做无法对这些资源进行依赖管理。而WebJars就提供给我们这些前端资源的jar包形势，我们就可以进行依赖管理。

3）引入jquery的依赖

```
<dependency>
    <groupId>org.webjars.bower</groupId>
    <artifactId>jquery</artifactId>
    <version>3.2.0</version>
</dependency>
```

4）创建index.html

在/src/java/resources/templates/目录下创建index.html文件，并在文件中引入jquery文件。这里我们的前端页面模板使用的是thymeleaf，并且在application.properties文件中配置了相应的模板前缀和后缀。

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>test webjars</title>
        <script src="/webjars/jquery/3.2.0/dist/jquery.min.js"></script>
    </head>
    <body>
        <h1></h1>
    </body>
    <script type="text/javascript">
        $(function(){
            $("h1").text("welcome use webJars!!!");
        });
    </script>
</html>
```

实际jar包中js资源查看

![](/images/spring/springboot/spring-webjars-jquery.png)

5）运行测试

```
mvn spring-boot:run
```

访问：http://localhost:8080