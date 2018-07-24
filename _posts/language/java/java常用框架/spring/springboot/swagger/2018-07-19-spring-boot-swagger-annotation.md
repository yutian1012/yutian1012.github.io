---
title: swagger注解类
tags: [spring]
---

参考：https://www.cnblogs.com/zq-boke/p/6423027.html

1）@Api注解（对接口分组）

提供的接口描述信息是以Controller进行分组的，该注解用于描述该Controller信息。

```
@Api(value = "描述", description = "控制器名称")
@RestController
public class SwaggerController {
    ...
}
```

注：在Controller类上使用该注解

2）@ApiOperation注解（具体接口描述，方法描述）

具体方法（接口）的描述信息，value指定接口名称，notes表示接口相信描述。

```
@ApiOperation(value = "接口名称", notes = "使用URL传参方式描述接口")
@RequestMapping(...)
public Customer api(int age,  String nickname) {
    ...
}
```

注：在Controller类的方法上使用该注解，实现对方法的描述

3）@ApiImplicitParam注解（方法参数描述）

该注解用于描述该接口所接受的参数信息。name指定参数名称，value指定参数介绍，paramType指定参数接收类型（如PathVariable变量，对应值为path），defaultValue指定在swaggerui界面中显示的默认值，dataType指定参数类型。

```
...
@ApiImplicitParams({
    @ApiImplicitParam(name = "age", value = "年龄", paramType = "path", 
        defaultValue = "2", dataType = "int"),
    @ApiImplicitParam(name = "nickname", value = "昵称", paramType = "query", 
        defaultValue = "张三", dataType = "string")
})
@RequestMapping(value = "api/{age}" ,...)
public Customer api(
    @PathVariable("age") int age, 
    @RequestParam("nickname") String nickname) {
    ...
}
```

如果方法中有多个参数时，可以使用@ApiImplicitParams注解，在注解中按照参数顺序定义参数描述信息。

注：Implicit中文意思有无疑的，绝对的意思。

注2：在Controller类的方法上使用该注解，实现对方法参数的描述

注3：一般用于描述基本类型的数据，对象型参数使用@ApiModel在实体类上单独描述

4）@ApiResponses注解（方法返回状态的描述）

该注解用于描述接口的返回内容。code指定HTTP返回状态，message指定状态描述，response返回类型

```
@ApiResponses({
    @ApiResponse(code = 200, message = "请求成功", response = Customer.class),
    @ApiResponse(code = 401, message = "权限不足"),
    @ApiResponse(code = 403, message = "禁止访问"),
    @ApiResponse(code = 404, message = "资源未找到"),
    @ApiResponse(code = 500, message = "程序内部发生错误")
})
@RequestMapping(value = "api/{age}", ...)
public Customer api(@PathVariable("age") int age,
    @RequestParam("nickname") String nickname) {
    ...
}
```

注：在Controller类的方法上使用该注解，实现对方法返回状态的描述

5）@ApiModel注解（用于方法中参数实体类的api描述，以及方法返回实体类的api描述）

参数实体：有时在方法中可以接收实体对象，该实体对象是有springmvc根据用户传递的参数自动封装到方法参数中的。

返回实体：有时在开发接口时会直接将对象返回到前端，如@ResponseBody将对象格式化成json数据返回。

```
@ApiModel(description = "顾客实体描述")
public class Customer {
    ...
}
```

controller方法不需要使用@ApiImplicitParam来描述

```
@ApiOperation(value = "接口名称", notes = "使用实体传参方式描述接口", response = Customer.class)
@RequestMapping(value = "api", method = RequestMethod.POST)
public Customer api(@RequestBody UserValidate userValidate) {
    ...
}
```

UserValidate和Customer实体都使用了@ApiModel注解

注：实体描述时也可以不使用@ApiModel，直接在实体属性上进行描述也是可以的。

注2：该注解用于实体类上

6）@ApiModelProperty注解（描述实体类中属性字段信息）

注解属性信息，value指定参数名称，example指定默认数据，required指定请求时该参数是否为必须的。

```
@ApiModel(description = "顾客实体描述")
public class Customer {

    @ApiModelProperty(value = "用户昵称", example = "Rico")
    private String nickname;

    @ApiModelProperty(value = "用户年龄", example = "18")
    private int age;
}
```

注：该注解用于实体类的属性上