---
title: github开源项目云收藏（用户登录）
tags: [project]
---

拷贝login.html页面到templates目录下

1）页面实现表单

```
<div class="panel-body" id="loginPage">
   <p class="text-center pv">请登录</p>
   <form id="form" data-parsley-validate="true" onsubmit="return false">

      <div class="form-group has-feedback">
         <input type="text" placeholder="邮箱地址或登录名称" v-model="username" class="form-control" data-parsley-error-message="请输入邮箱地址或登录名称" required="required" />
         <span class="fa fa-envelope form-control-feedback text-muted"></span>
      </div>

      <div class="form-group has-feedback">
         <input type="password" placeholder="密码" v-model="password" class="form-control" data-parsley-error-message="请输入密码" required="required" />
         <span class="fa fa-lock form-control-feedback text-muted"></span>
      </div>

      <div class="clearfix">
         <div class="pull-right">
            <a href="/forgotPassword" class="text-muted">忘记密码？</a>
         </div>
      </div>

      <button v-on:click="login" class="btn btn-block btn-primary mt-lg">登录</button>

   </form>
</div>
```

注：可以看到这里校验使用的是parsley，绑定和提交事件使用的是vue。

2）js处理代码

```
<script th:src="@{/webjars/jquery/2.0.0/jquery.min.js}"></script>
<script th:src="@{/webjars/vue/1.0.24/vue.min.js}"></script>  
<script th:src="@{/webjars/vue-resource/0.7.0/dist/vue-resource.min.js}"></script>
<script th:src="@{/vendor/parsleyjs/dist/parsley.min.js}"></script> 
<script type='text/javascript'>
Vue.http.options.emulateJSON = true; 
var loginPage = new Vue({//绑定页面div的id为loginPage元素
    el: '#loginPage',
    data: { //定义变量与页面的input域双向绑定
        'username': '',
        'password': ''
    },
    methods: {
        login: function(event){ //定义登录按钮事件
            var ok = $('#form').parsley().isValid({force: true});
            if(!ok){
                return;
            }
            var datas={//提交数据
                     userName: this.username,
                     passWord: this.password
                     };
            //请求登录操作
            this.$http.post('/user/login',datas).then(function(response){
                 if(response.data.rspCode == '000000'){
                         window.open(response.data.url, '_self');
                 }else{
                      $("#errorMsg").html(response.data.rspMsg);
                      $("#errorMsg").show();
                 }
             }, function(response){
                 console.log('error');
             })
        }
    }
})
</script>
```

3）controller类处理登录逻辑

在UserController类添加login方法，处理登录逻辑

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
        //这里不是bug，前端userName有可能是邮箱也有可能是昵称。
        User loginUser = userService
            .findByUserNameOrEmail(user.getUserName(), user.getUserName());
        if (loginUser == null ) {
            return new Response(ExceptionMsg.LoginNameNotExists);
        }else if(!loginUser.getPassWord().equals(user.getPassWord())){
            return new Response(ExceptionMsg.LoginNameOrPassWordError);
        }
        
        String preUrl = "/";
        
        Response res=new Response(ExceptionMsg.SUCCESS);
        res.setUrl(preUrl);
        
        return res;
    } catch (Exception e) {
        logger.error("login failed, ", e);
        return new Response(ExceptionMsg.FAILED);
    }
}
```

注：在Response对象中添加了一个url属性变量，用来设置成功登录后的跳转路径

4）添加登录后的处理方法

修改IndexController类，添加home方法，

```
/**
 * 登录后的主页面
 * @param model
 * @return
 */
@RequestMapping(value="/",method=RequestMethod.GET)
public String home(Model model) {
    return "home";
}
```

5）home.html页面

```
<html xmlns:th="http://www.thymeleaf.org" data-layout-decorator="fragments/layout">

  <head th:include="fragments/layout :: htmlhead" th:with="title='favorites'"></head>

  <body>

      <section data-layout-fragment="content">

      </section>

  </body>

  <script type='text/javascript'>

 </script>

</html>
```