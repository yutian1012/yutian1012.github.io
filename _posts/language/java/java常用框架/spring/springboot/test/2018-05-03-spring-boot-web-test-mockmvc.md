---
title: spring boot MockMvc
tags: [spring]
---

参考：https://blog.csdn.net/amethyst128/article/details/73332150

参考：https://www.cnblogs.com/0201zcr/p/5756642.html

Mock中文意思：模拟的; 仿制的; 虚假的; 不诚实的。

MockMvc表示模拟MVC用来测试mvc的流转过程。

1）测试类代码

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