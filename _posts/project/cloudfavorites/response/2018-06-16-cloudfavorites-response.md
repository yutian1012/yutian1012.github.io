---
title: github开源项目云收藏（前后台统一返回对象）
tags: [project]
---

后台对响应信息封装到Response对象中

1）Response对象的定义

```
package common;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Response {
    /** 返回信息码*/
    private String rspCode="000000";
    /** 返回信息内容*/
    private String rspMsg="操作成功";
}
```

2）后台处理代码返回Response对象

在controllerr类上使用@RestController注解，并返回Response对象即可

3）前端接收响应的处理

```
//提交json数据到后台
var datas={
 email: this.email,
 passWord: this.password,
 userName: this.userName
 };
//vue框架发生http请求，并处理响应数据
this.$http.post('/user/regist',datas)
    .then(function(response){//vue将返回对象封装到data中
     if(response.data.rspCode == '000000'){
          window.open('/', '_self');
     }else{
         $("#errorMsg").html(response.data.rspMsg);
         $("#errorMsg").show();
     }
 }, function(response){
     console.log('error');
 })
```