---
title: spring的controller层的异常捕获
tags: [spring]
---

1）问题

访问系统controller的某个方法时抛出异常

```
org.springframework.web.bind.MissingServletRequestParameterException: Required Long parameter 'userId' is not present
```

注：该异常没有打印出过多的信息，现在想要捕获该异常，并查看具体的堆栈信息，从而定位错误。

2）在controller中配置异常处理类

```
@ExceptionHandler 
public void exception(HttpServletRequest request, Exception ex) {        
    ex.printStackTrace();
}  
```

注：定义该类所抛出异常的捕获方法

3）问题发现

```
@ModelAttribute
protected SysUser getFormObject(@RequestParam("userId") Long userId,    Model model) throws Exception{
    
}
```

注：在controller类中发现了@ModelAttribute注解，在调用controller中的每个方法时，都会从参数中获取userId，如果不提供userId参数，则抛出异常。