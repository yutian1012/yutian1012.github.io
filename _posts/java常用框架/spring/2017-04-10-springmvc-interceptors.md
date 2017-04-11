---
title: spring mvc拦截器
tags: [spring]
---

参考：http://elim.iteye.com/blog/1750680

参考：http://www.cnblogs.com/yangzhilong/p/3725849.html

方案一，（近似）总拦截器，拦截所有url

```
<mvc:interceptors>
    <bean class="com.app.mvc.MyInteceptor" />
</mvc:interceptors>
```

为什么叫“近似”，前面说了，Spring没有总的拦截器。

mvc:interceptors会为每一个HandlerMapping，注入一个拦截器。总有一个HandlerMapping是可以找到处理器的，最多也只找到一个处理器，所以这个拦截器总会被执行的。起到了总拦截器的作用。

如果是REST风格的URL，静态资源也会被拦截。（在4.0上测试并未有此问题）
方案二， （近似） 总拦截器， 拦截匹配的URL。

```
<mvc:interceptors >  
  <mvc:interceptor>  
        <mvc:mapping path="/user/*" /> <!-- /user/*  -->  
        <bean class="com.mvc.MyInteceptor"></bean>  
    </mvc:interceptor>  
</mvc:interceptors>  
```
就是比 方案一多了一个URL匹配。

如果是REST风格的URL，静态资源也会被拦截。（在4.0上测试并未有此问题）
方案三,HandlerMappint上的拦截器。

如果是REST风格的URL，静态资源就不会被拦截。因为我们精准的注入了拦截器
复制代码
```
<bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping">     
 <property name="interceptors">     
     <list>     
         <bean class="com.mvc.MyInteceptor"></bean>    
     </list>     
 </property>     
</bean> 
```

Spring拦截器的定义：

Spring为我们提供了：org.springframework.web.servlet.HandlerInterceptor接口，

org.springframework.web.servlet.handler.HandlerInterceptorAdapter适配器，
实现这个接口或继承此类，可以非常方便的实现自己的拦截器。
 
有以下三个方法：
 
```
//Action之前执行:
 public boolean preHandle(HttpServletRequest request,
   HttpServletResponse response, Object handler);
//如果返回false则中断请求
 
//生成视图之前执行
 public void postHandle(HttpServletRequest request,
   HttpServletResponse response, Object handler,
   ModelAndView modelAndView);
 
//最后执行，可用于释放资源
 public void afterCompletion(HttpServletRequest request,
   HttpServletResponse response, Object handler, Exception ex)
```
