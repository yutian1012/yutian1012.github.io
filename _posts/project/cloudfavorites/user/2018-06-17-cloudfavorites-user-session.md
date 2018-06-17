---
title: github开源项目云收藏（session）
tags: [project]
---

系统必须能够判断用户是否登录，如果用户没有登录则需要跳转到相应的login页面，登录系统。如果已登录了系统，那么可以从session中获取用户的信息。

1）创建安全过滤器来判断

该类实现javax.servlet.Filter接口

```
package com.favorites.favoriteswebdemo.filter;

import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.favorites.favoriteswebdemo.common.Const;

/**
 * 安全访问过滤器
 */
public class SecurityFilter implements Filter{
    protected Logger logger =  LoggerFactory.getLogger(this.getClass());
    //可以匿名访问的连接地址
    private static Set<String> GreenUrlSet = new HashSet<String>();
    
    @Override
    public void init(FilterConfig config) throws ServletException {
        GreenUrlSet.add("/login");
        GreenUrlSet.add("/register");
        GreenUrlSet.add("/index");
        GreenUrlSet.add("/forgotPassword");
        GreenUrlSet.add("/newPassword");
        GreenUrlSet.add("/tool");
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        String uri = req.getRequestURI();
        if (req.getSession().getAttribute(Const.LOGIN_SESSION_KEY) == null) {
            //判断访问的地址是否允许匿名访问
            if (containsSuffix(uri)  || GreenUrlSet.contains(uri)) {
                logger.debug("don't check  url , " + uri);
                chain.doFilter(request, response);
                return;
            }
            //请求转发到登录页面
            req.getRequestDispatcher("/login").forward(request,response);
        }
        
        chain.doFilter(request, response);
    }
    
    /**
     * 过滤静态资源
     */
    private boolean containsSuffix(String url) {
        if (url.endsWith(".js")
                || url.endsWith(".css")
                || url.endsWith(".jpg")
                || url.endsWith(".gif")
                || url.endsWith(".png")
                || url.endsWith(".html")
                || url.endsWith(".eot")
                || url.endsWith(".svg")
                || url.endsWith(".ttf")
                || url.endsWith(".woff")
                || url.endsWith(".ico")
                || url.endsWith(".woff2")) {
            return true;
        } else {
            return false;
        }
    }

    @Override
    public void destroy() {
    }
}
```

2）配置spring boot加载过滤器

```
package com.favorites.favoriteswebdemo;

import javax.servlet.Filter;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.favorites.favoriteswebdemo.filter.SecurityFilter;

@Configuration
public class WebConfiguration {
    //配置filter过滤器
     @Bean
     public FilterRegistrationBean<Filter> filterRegistration() {
         FilterRegistrationBean<Filter> registration = new FilterRegistrationBean<Filter>();
         registration.setFilter(new SecurityFilter());
         registration.addUrlPatterns("/*");
         //registration.addInitParameter("paramName", "paramValue");
         registration.setName("MyFilter");
         registration.setOrder(1);
         return registration;
     }
}
```

3）修改登录逻辑

在用户成功登录后，将用户信息存放到session中。修改UserController类的login方法

```
/**
 * 登录
 * @param user
 * @param response
 * @return
 */
@RequestMapping(value = "/login", method = RequestMethod.POST)
public Response login(User user,HttpServletResponse response) {
    try {
        //用户登录逻辑
        ....
        
        //登录成功保存数据到session中
        saveSession(loginUser);
        
        return res;
    } catch (Exception e) {
        ...
    }
}
/**
 * 将登录用户信息保存到session中
 * @param user
 */
public void saveSession(User user) {
    HttpServletRequest request=((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
    request.getSession().setAttribute(Const.LOGIN_SESSION_KEY, user);
}
```

4）测试

访问用户首页会自动跳转到登录页面，用户成功登录后才可以访问用户的首页。