---
title: 使用dubbo搭建微服务架构-配置网关-配置权限拦截器
tags: [architecture]
---

dubbo中所暴露的接口由网关服务转为http协议的api调用，而这些接口通常需要安全检查，安全检查的实现可通过拦截器的方式来实现。

实现方式：在spring-boot中配置拦截器，通过拦截请求中是否带有token，来决定是否放行请求。

1）定义权限拦截器

```
package org.sjq.dubbo.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class RequestInterceptor implements HandlerInterceptor{
    /**
     * 前置处理
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object obj) throws Exception {
        String token=request.getParameter("token");
        if(token!=null&&token.equals("1")) {
            return true;
        }
        response.getWriter().write("token error");
        return false;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object obj, Exception exception)
            throws Exception {
        System.out.println("invoke aftetCompletion method");
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object object, ModelAndView mv)
            throws Exception {
        System.out.println("invoke postHandle");
    }
}
```

2）配置拦截器

创建一个@Configuration类并配置相应的拦截器

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
        
        registry.addInterceptor(new RequestInterceptor()).addPathPatterns("/**");
        
        super.addInterceptors(registry);
    }

}
```

3）访问测试

```
# 不带token参数，访问被拒绝
http://localhost:8081
# 带token参数，能够通过权限校验
http://localhost:8081?token=1
```