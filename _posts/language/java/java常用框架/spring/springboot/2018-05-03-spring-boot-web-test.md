---
title: spring boot web测试
tags: [spring]
---

### 测试代码

1）测试controller方法

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

2）测试类

```
package com.example.demo.controller;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultHandlers;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class HelloWorldControlerTests {

    private MockMvc mvc;
    
    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new HelloWorldController()).build();
    }
   
    @Test
    public void getHello() throws Exception {
    mvc.perform(MockMvcRequestBuilders.get("/hello")
        .accept(MediaType.APPLICATION_JSON))
        .andExpect(MockMvcResultMatchers.status().isOk())
        .andDo(MockMvcResultHandlers.print())
        .andReturn();
    }
}
```

### 测试代码分析

1）RunWith注解分析

```
@RunWith就是一个运行器

@RunWith(JUnit4.class)就是指用JUnit4来运行

@RunWith(SpringJUnit4ClassRunner.class)让测试运行于Spring测试环境。即使用了Spring的SpringJUnit4ClassRunner，以便在测试开始的时候自动创建Spring的应用上下文。

@RunWith(Suite.class)的话就是一套测试集合。
```

2）SpringBootTest注解分析

该注解可以在springboot中引入相关的测试类，而且可以进行环境变量及属性的设置。

a.class属性

classes参数表示将某些类纳入测试环境的容器中。

```
@SpringBootTest(classes = MockServletContext.class)
```

b.properties属性

properties参数设置配置信息，除了在配置文件中设置属性，properties也可以设置相关属性。

```
@SpringBootTest(properties = {"app.version=1.0"})
```

3）设置环境属性

使用EnvironmentTestUtils.addEnvironment方法进行环境属性设置，一般在测试类的before注解上设置环境属性值。

```
@Autowired
private ConfigurableEnvironment environment;

@Before
public void init(){
    EnvironmentTestUtils.addEnvironment(environment,"app.admin.user=zhangsan");
}
```

4）WebAppConfiguration注解

该注解是用来声明加载的ApplicationContex是一个WebApplicationContext，它的属性指定的是Web资源的位置。默认为src/main/webapp。

```
@WebAppConfiguration("src/main/resources") : 
```