---
title: spring的注解方式的校验
tags: [spring]
---

参考：http://www.cnblogs.com/liukemng/p/3738055.html

### 注意事项

1）@Valid注解

```
import javax.validation.Valid;
```

该注解的作用是标识该实体类需要进行校验，然后根据实体对象的属性上中使用的注解进行校验。

2）controller方法上实现校验

```
public String test(Model model, 
    @Valid ValidateModel validateModel, BindingResult result){
    ...
}
```

注：BindingResult必须紧跟在校验的实体对象后，否则抛出异常

```
nested exception is java.lang.IllegalStateException: Errors/BindingResult argument declared without preceding model attribute.
```