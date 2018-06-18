---
title: github开源项目云收藏（全局异常处理）
tags: [project]
---

参考：https://www.cnblogs.com/junzi2099/p/7840294.html

参考2：https://blog.csdn.net/butioy_org/article/details/78718405

参考3：https://blog.csdn.net/github_38659047/article/details/72930369

1）处理404异常

spring mvc当找不到处理方法时，默认会跳转到error页面，并不会抛出异常供全局异常处理类处理。

```
spring.mvc.throw-exception-if-no-handler-found=true
spring.resources.add-mappings=false
```

2）创建全局异常处理类

```
package com.favorites.favoriteswebdemo.exception;

import org.apache.catalina.servlet4preview.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.servlet.ModelAndView;

import com.favorites.favoriteswebdemo.common.ExceptionMsg;
import com.favorites.favoriteswebdemo.common.Response;
import com.favorites.favoriteswebdemo.view.JacksonView;

@ControllerAdvice
public class GlobalExceptionHandler {

    protected Logger logger =  LoggerFactory.getLogger(this.getClass());

    public static final String DEFAULT_ERROR_VIEW = "testerror";

    @ExceptionHandler(value = Exception.class)
    @ResponseStatus(code=HttpStatus.INTERNAL_SERVER_ERROR)
    public ModelAndView defaultErrorHandler(Exception e, HttpServletRequest request) throws Exception {
        logger.info("请求地址：" + request.getRequestURL());
        //rlogger.error("异常信息：",e);
        
        ModelAndView mav = new ModelAndView();
        
        if (isAjax(request)) {
            JacksonView view = new JacksonView();
            view.addStaticAttribute("data", new Response(ExceptionMsg.FAILED));
            mav.setView(view);
        } else {
            mav.setViewName(DEFAULT_ERROR_VIEW);
        }
        
        return mav;
    }
    /**
     * 判断请求是否为ajax请求
     * @param request
     * @return
     */
    private boolean isAjax(HttpServletRequest request) {
        return "XMLHttpRequest".equalsIgnoreCase(request.getHeader("X-Requested-With"));
    }
}
```

3）创建jackson视图

```
package com.favorites.favoriteswebdemo.view;

import java.io.ByteArrayOutputStream;
import java.util.Map;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.view.AbstractView;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JacksonView extends AbstractView{

    @Override
    protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request,
            HttpServletResponse response) throws Exception {

        try(ByteArrayOutputStream outnew = new ByteArrayOutputStream();ServletOutputStream out = response.getOutputStream();){
            ObjectMapper mapper = new ObjectMapper();  
            String json = mapper.writeValueAsString(model);  
            
            int len=json.getBytes().length;
            
            outnew.write(json.getBytes());
            
            response.setContentLength(len);
            
            outnew.writeTo(out);
            outnew.close();
            out.flush();
        };
    }
}
```