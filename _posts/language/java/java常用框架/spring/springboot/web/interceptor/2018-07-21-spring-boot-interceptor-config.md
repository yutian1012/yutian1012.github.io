---
title: spring boot拦截器配置
tags: [spring]
---

创建一个@Configuration类并配置相应的拦截器，WebMvcConfigurerAdapter类中定义了web相关的注入接口，通过该适配器提供的方法，能够注册相应的组件到web环境中。

1）配置拦截器

```
package org.sjq.dubbo.configuration;

import org.sjq.dubbo.interceptor.RequestInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class WebAppConfig extends WebMvcConfigurerAdapter{
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        
        registry.addInterceptor(new RequestInterceptor())
            .addPathPatterns("/**");
        
        super.addInterceptors(registry);
    }

}
```

2）InterceptorRegistry类实现拦截器设置

该类提供了addInterceptor方法用于添加自定义拦截器，并通过addPathPatterns方法设置拦截器的拦截路径。

3）WebMvcConfigurerAdapter类实现组件注入web环境

super.addInterceptors(registry)，该方法将相关的拦截器注册信息配置到web环境下。

4）拦截路径

/** 会匹配拦截所有路径

/account/** 只会拦截请求路径中以/account/开头的所有请求