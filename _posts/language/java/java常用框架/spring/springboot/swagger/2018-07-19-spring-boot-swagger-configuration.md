---
title: swagger配置类
tags: [spring]
---

swagger配置，可通过读取pom.xml获取项目的version等信息

1）引入相关的依赖

```
<dependency>
    <groupId>org.apache.maven</groupId>
    <artifactId>maven-model</artifactId>
    <version>3.5.3</version>
</dependency>
```

maven-model用于解析maven的pom文件中的内容

2）配置类

```
package org.book.rpc.dubbo.swagger.configuration;

import org.apache.maven.model.Model;
import org.apache.maven.model.io.xpp3.MavenXpp3Reader;
import org.codehaus.plexus.util.xml.pull.XmlPullParserException;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

@Configuration
@EnableSwagger2
public class Swagger2Configuration {

    @Bean
    public Docket createRestApi() throws IOException, XmlPullParserException {
        MavenXpp3Reader reader = new MavenXpp3Reader();
        Model model = reader.read(new FileReader("pom.xml"));

        ApiInfo apiInfo = new ApiInfoBuilder()
                .title("接口名称")
                .description("接口介绍内容")
                .version(model.getVersion())
                .build();

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.demo.controller"))
                .paths(PathSelectors.any())
                .build();
    }
}
```

3）相关类介绍

MavenXpp3Reader读取maven的pom文件，并封装到model对象中。

ApiInfo用于配置该应用所产生的接口基本描述信息。

Docket用于构建swagger接口描述文档，并通过apis方法指定接口所在的位置。