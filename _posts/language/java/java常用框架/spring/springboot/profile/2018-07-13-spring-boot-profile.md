---
title: spring boot多个配置文件
tags: [spring]
---

参考：https://www.cnblogs.com/winner-0715/p/6754994.html

针对单一配置文件的问题，spring boot开发的项目，本地开发环境一般与测试环境和运行环境是不同的，因此在部署系统时经常需要修改配置信息，很容易造成错漏。

spring boot提供了不同环境不同配置文件的功能。

1）环境与配置文件的关系

开发环境：dev，对应配置文件为：application-dev.yml

测试环境：test，对应配置文件为：application-test.yml

运行环境：prd，对应配置文件为：application-prd.yml

2）激活配置文件

方式一：在application.yml中激活

```
spring:
    profiles:
        active: dev
```

方式二：运行时指定

```
java -jar myapp.jar --spring.profiles.active=dev
```

3）单元测试激活方式

```
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("dev")
public class XxxxxTest {
    ...
}
```

4）在application.yml中同时写多个环境的配置信息

使用三个短杠分隔多环境

```
# 上方配置全局环境
....

# 使用---，分隔多个环境的配置信息
---
spring:
  profiles: dev
  
server:
  port: 8088
  
---
spring:
  profiles: test
  
server:
  port: 8081

```

5）配置信息的覆盖问题

在application.yml文件中配置一个自定义的属性

```
info:
 app.name: spider
```

在application-dev.yml中配置相同的属性

```
info: # 测试覆盖问题
 app.name: spider-dev
```

测试代码

```
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("dev")
public class ProfileTest {

    @Value("${info.app.name}")
    private String appName;
    
    @Test
    public void testProfile() throws ParseException {
        System.out.println(appName);
    }
}
```

注：结果输出为spider-dev，如果去掉application-dev.yml中定义的属性，则输出spider。