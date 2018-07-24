---
title: swagger注解类实例
tags: [spring]
---

1）接口api描述

```
@Api(value = "描述", description = "控制器名称")
@RestController
public class SwaggerController {

    @ApiOperation(value = "接口名称", notes = "使用URL传参方式描述接口")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "age", value = "年龄", paramType = "path", defaultValue = "2", dataType = "int"),
            @ApiImplicitParam(name = "nickname", value = "昵称", paramType = "query", defaultValue = "张三", dataType = "string")
    })
    @ApiResponses({
            @ApiResponse(code = 200, message = "请求成功", response = Customer.class),
            @ApiResponse(code = 401, message = "权限不足"),
            @ApiResponse(code = 403, message = "禁止访问"),
            @ApiResponse(code = 404, message = "资源未找到"),
            @ApiResponse(code = 500, message = "程序内部发生错误")
    })
    @RequestMapping(value = "api/{age}", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
    public Customer api(@PathVariable("age") int age, @RequestParam("nickname") String nickname) {
        Customer customer = new Customer();
        customer.setAge(age);
        customer.setNickname(nickname);
        return customer;
    }

    @ApiOperation(value = "接口名称", notes = "使用实体传参方式描述接口", response = Customer.class)
    @RequestMapping(value = "api", method = RequestMethod.POST)
    public Customer api(@RequestBody UserValidate userValidate) {
        Customer customer = new Customer();
        customer.setAge(userValidate.getAge());
        customer.setNickname(userValidate.getNickname());
        return customer;
    }

}
```

2）实体api描述

实例中返回实体类的api描述

```
package org.book.rpc.dubbo.swagger.entity;

import io.swagger.annotations.ApiModelProperty;

public class Customer {

    @ApiModelProperty(value = "用户昵称", example = "Rico")
    private String nickname;

    @ApiModelProperty(value = "用户年龄", example = "18")
    private int age;
}
```

实例中参数实体类的api描述

```
package org.book.rpc.dubbo.swagger.validate;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel(description = "参数描述")
public class UserValidate {

    @ApiModelProperty(value = "用户昵称", required = true, example = "张三")
    private String nickname;

    @ApiModelProperty(value = "用户年龄", required = true, example = "20")
    private int age;
}
```