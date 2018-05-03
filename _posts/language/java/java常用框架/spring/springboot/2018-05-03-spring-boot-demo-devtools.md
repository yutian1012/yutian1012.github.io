---
title: spring boot devtools实现热部署
tags: [spring]
---

参考：https://blog.csdn.net/isea533/article/details/70495714

spring-boot-devtools模块可以包含在任何项目中，目的是在开发中能够针对代码的修改自动重启服务，清空服务器端的缓存等，让开发人员能够理解查看到修改的内容。它可以节省大量的时间。

1）devtools的作用

a.开发环境下禁用某些模块的缓存

Spring Boot 支持的一些库中会使用缓存来提高性能。例如模版引擎将缓存编译后的模板，以避免重复解析模板文件。虽然缓存在生产中非常有益，但它在开发过程中可能会产生反效果，它会阻止你看到刚刚在应用程序中进行的更改。

注：spring-boot-devtools将默认禁用这些缓存选项。

b.自动重启（通过监视变动实现）

spring-boot-devtools会在类路径上的文件发生更改时自动重启。这在IDE中工作时可能是一个有用的功能，因为它为代码更改提供了非常快的反馈循环。 默认情况下会监视类路径上的所有变动，但请注意，某些资源（如静态资源和视图模板）不需要重启应用程序。

2）排除不需要监控的资源变动

某些资源在更改时不一定需要触发重启。如直接编辑Thymeleaf模板。默认情况下，更改/META-INF/maven，/META-INF/resources，/resources，/static，/public或/templates中的资源不会触发重启，但会触发实时重新加载。

如果要自定义这些排除项，可以使用spring.devtools.restart.exclude属性。

如果你想保留上面的默认（情况下的）值并添加其他的排除项，你可以使用spring.devtools.restart.additional-exclude属性。

编辑application.properties文件，多个值使用逗号分隔

```
spring.devtools.restart.exclude=static/**,public/**

spring.devtools.restart.additional-exclude=...
```

3）监控额外的路径

当对不在类路径中的文件进行更改时，可能需要重启或重新加载应用程序。为此，请使用spring.devtools.restart.additional-paths属性来配置监视其他路径的更改。

```
spring.devtools.restart.additional-paths=...
```

4）配置添加依赖

将依赖标记为optional可选是一种最佳做法，可以防止将devtools依赖传递到其他模块中。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    <scope>true</scope>
</dependency>
```

注：运行打包的应用程序时，开发人员工具会自动禁用。如果你通过java-jar或者其他特殊的类加载器进行启动时，都会被认为是生产环境的应用。

5）测试

```
package com.example.demo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {   
 
    @RequestMapping("/hello")    
    public String index() { 
        return "Hello World";
    }
}
```

启动服务

```
mvn spring-boot:run
```

访问：http://localhost:8080/hello

5）测试修改方法

当前HelloWorldController类的index方法返回Hello World，修改返回值返回Hello。

```
@RestController
public class HelloWorldController {   
 
    @RequestMapping("/hello")    
    public String index() { 
        return "Hello";
    }
}
```

再次访问，查看是否返回值发生了变化：http://localhost:8080/hello

注：这里不需要再次启动服务，直接访问已启动的服务。

6）测试添加新方法

HelloWorldController类中添加一个新的test方法

```
@RequestMapping("/test")
public String test() {
    return "test";
}
```

访问：http://localhost:8080/test