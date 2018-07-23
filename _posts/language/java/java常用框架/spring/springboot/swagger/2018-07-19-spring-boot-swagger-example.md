---
title: swagger入门实例
tags: [spring]
---

参考：https://blog.csdn.net/saytime/article/details/74937664

1）新建maven项目并引入依赖

```
<groupId>org.sjq.test</groupId>
<artifactId>swagger-demo</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>
```

依赖

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.8.0</version>
    </dependency>
     
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.8.0</version>
    </dependency>

    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-model</artifactId>
        <version>3.5.3</version>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

maven-model，用于解析maven的pom文件中的内容

springfox-swagger2用于生成接口描述文档的核心依赖

springfox-swagger-ui用于解析接口描述的界面依赖。

3）创建配置文件application.yml

```
server:
 port: 8003
```


4）创建启动类

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import springfox.documentation.swagger2.annotations.EnableSwagger2;

@SpringBootApplication
@EnableSwagger2
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

注：添加注解@EnableSwagger2

5）创建swagger配置类

```
package com.example.demo.conf;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
public class SwaggerConfiguration {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.demo.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    
    //配置接口描述信息
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("springboot利用swagger构建api文档")
                .description("简单优雅的restful风格")
                .version("1.0") //可通过解析pom文件获取version信息，这里暂时写死
                .build();
    }
}
```

6）创建controller类

```
package com.example.demo.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiOperation;

@Api("接口测试")
@RestController
public class HelloController {
    private Logger logger=LoggerFactory.getLogger(UserController.class);
    
    @RequestMapping(value="/hello")
    @ApiOperation(value="访问接口",notes="传递消息访问接口")
    @ApiImplicitParam(name="message",value="消息信息",dataType="String")
    public String hello(String message) {
        
        logger.info("接收客户端传递的数据..."+message);
        
        return "helloworld!!!";
    }
}
```

7）查看生成的接口描述

启动服务类DemoApplication，访问页面：localhost:8003/swagger-ui.html

![](/images/spring/springboot/swagger/swagger-controller.png)

8）操作过程

点击try it out，会要求编辑参数信息，下方有execute按钮用来发送消息。

![](/images/spring/springboot/swagger/swagger-method-execute.png)