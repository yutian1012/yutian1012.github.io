---
title: swagger与@RequestMapping属性的关系
tags: [spring]
---

在@RequestMapping中的属性可以用来限定方法的访问（method属性），接收的数据类型（consumes属性），返回的数据类型（produces属性）

```
@RequestMapping(value = "api/{age}", 
    method = RequestMethod.GET, 
    produces = MediaType.APPLICATION_JSON_VALUE)
public Customer api(@PathVariable("age") int age, @RequestParam("nickname") String nickname) {
    ...
}
```

1）@RequestMapping的属性与swagger-ui的关系如何？