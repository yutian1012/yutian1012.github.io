---
title: spring boot拦截器
tags: [spring]
---

spring-boot拦截器，用于拦截用户请求，进行特殊的处理。可通过配置拦截地址设置多个拦截器，这些拦截器如果拦截同一个请求，则组成一个拦截器链。

1）定义权限拦截器

拦截器类需要实现HandlerInterceptor接口，该接口定义了三个方法preHandle，afterCompletion，postHandle。

方法一：prehandle方法在请求处理之前执行，主要用于权限验证，参数过滤等。

注：该方法返回true或false，但返回true时会继续执行下一个拦截器，直到所有的拦截器都运行完，再运行被拦截的controller。返回false时不再执行后续的拦截器链及被拦截的controller。

方法二：postHandle方法，当前请求进行处理之后执行，主要用于日志记录、权限检查、性能监控、通用行为等。

方法三：afterCompletiong方法，当对应的preHandle方法的返回值为true时执行，该方法将在整个请求结束之后，也就是在DispatcherServlet渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。

2）拦截器的一个应用场景

拦截器用于权限过滤，通过拦截请求中是否带有token，来决定是否放行请求。

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

3）拦截器方法的执行顺序

可配置一个contorller类，并使用该拦截器拦截对该controller的请求，查看方法执行的顺序。

这里直接给出结论：

prehandle-->controller方法-->postHandle-->afterCompletion