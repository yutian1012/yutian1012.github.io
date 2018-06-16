---
title: github开源项目云收藏（用户注册）
tags: [project]
---

将原项目的static目录下的文件拷贝到测试项目下，并将register.html拷贝到templates下。

1）创建User对象

```
package com.favorites.favoriteswebdemo.domain;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Getter
@Setter
@NoArgsConstructor
public class User implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false, unique = true)
    private String userName;
    @Column(nullable = false)
    private String passWord;
    @Column(nullable = false, unique = true)
    private String email;   
    @Column(nullable = true)
    private String profilePicture;
    @Column(nullable = true,length = 65535,columnDefinition="Text")
    private String introduction;
    @Column(nullable = false)
    private Long createTime;
    @Column(nullable = false)
    private Long lastModifyTime;
    @Column(nullable = true)
    private String outDate;
    @Column(nullable = true)
    private String validataCode;
    @Column(nullable = true)
    private String backgroundPicture;
}
```

2）跳转注册页面

在IndexController类中添加regist方法，用于跳转regist注册页面

```
package com.favorites.favoriteswebdemo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class IndexController {
    @RequestMapping(value="/register",method=RequestMethod.GET)
    public String regist() {
        return "register";
    }
}
```

3）创建注册页面

在templates目录下创建register.html页面，页面使用了parsleyjs进行表单校验，同时使用vue进行表单变量绑定和点击事件处理。

```
<div class="panel-body" id="registPage">
   <p class="text-center pv">快速注册新用户</p>
   <form id="form" data-parsley-validate="true" onsubmit="return false">

      <div class="form-group has-feedback">
         <label for="signupInputEmail1" class="text-muted">登录邮箱</label>
         <input id="signupInputEmail1" type="email" placeholder="输入邮箱地址" class="form-control" v-model="email"  data-parsley-required-message="请输入邮箱地址" data-parsley-type-message="请输入正确的邮箱地址" required="required"/>
         <span class="fa fa-envelope form-control-feedback text-muted"></span>
      </div>

      <div class="form-group has-feedback">
         <label for="signupInputName" class="text-muted">登录名称</label>
         <input id="signupInputName" type="text" placeholder="我们该如何称呼您？" class="form-control" v-model="userName" data-parsley-required-message="请输入登录名称" required="required"/>
         <span class="fa fa-user form-control-feedback text-muted"></span>
      </div>

      <div class="form-group has-feedback">
         <label for="signupInputPassword" class="text-muted">密码</label>
         <input id="signupInputPassword" type="password" placeholder="密码" class="form-control" v-model="password" required="required" data-parsley-required-message="请输入密码" pattern="/^(?![0-9]+$)(?![a-zA-Z]+$)(?![^0-9a-zA-Z]+$)\S{6,20}$/" data-parsley-pattern-message="请输入6-20位字母数字组合" />
         <span class="fa fa-lock form-control-feedback text-muted"></span>
      </div>

      <button v-on:click="regist" 
            class="btn btn-block btn-primary mt-lg">创建账户</button>
   </form>
</div>
```

注：上面使用了parsley进行表单校验。

访问下面地址，直接跳转到注册页面

```
http://localhost:9090/register
```

4）js实现注册校验与提交

这里使用vue来进行页面数据的绑定与事件处理

注：pom中已经引入了相关的webjars依赖了，这里直接引用相关的依赖即可

```
<script th:src="@{/webjars/jquery/2.0.0/jquery.min.js}"></script>
<script th:src="@{/webjars/vue/1.0.24/vue.min.js}"></script>   
<script th:src="@{/webjars/vue-resource/0.7.0/dist/vue-resource.min.js}"></script>
<script th:src="@{/vendor/parsleyjs/dist/parsley.min.js}"></script> 
<script type='text/javascript'>
    Vue.http.options.emulateJSON = true; 
    var registPage = new Vue({
        el: '#registPage', //指定待绑定的dom的ID属性
        data: { //定义变量，页面input使用v-model="email"来双向绑定
            'email': '', 
            'password': '',
            'userName':''
        },
        methods: { //定义相关事件，button控件绑定提交事件v-on:click="regist" 
            regist: function(event){
                var ok = $('#form').parsley().isValid({force: true});
                if(!ok){
                    return;
                }
                var datas={
                         email: this.email,
                         passWord: this.password,
                         userName: this.userName
                         };
                //提交数据到请求地址/user/regist
                this.$http.post('/user/regist',datas).then(function(response){
                     if(response.data.rspCode == '000000'){
                          window.open('/', '_self');
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

5）创建后台提交处理类

创建UserController类，并创建regist方法来处理注册逻辑

```
package com.favorites.favoriteswebdemo.controller;

import javax.annotation.Resource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import com.favorites.favoriteswebdemo.common.ExceptionMsg;
import com.favorites.favoriteswebdemo.common.Response;
import com.favorites.favoriteswebdemo.domain.User;
import com.favorites.favoriteswebdemo.service.UserService;

@RestController
@RequestMapping("/user")
public class UserController {
    
    @Resource
    private UserService userService;
    
    protected Logger logger =  LoggerFactory.getLogger(this.getClass());
    
    @RequestMapping(value = "/regist", method = RequestMethod.POST)
    public Response regist(User user) {
        try {
            User registUser = userService.findByEmail(user.getEmail());
            if (null != registUser) {
                return result(ExceptionMsg.EmailUsed);
            }
            User userNameUser = userService.findByUserName(user.getUserName());
            if (null != userNameUser) {
                return result(ExceptionMsg.UserNameUsed);
            }
            //user.setPassWord(getPwd(user.getPassWord()));//暂时存储为明文，后期可以对其进行加密处理
            user.setCreateTime(System.currentTimeMillis());
            user.setLastModifyTime(System.currentTimeMillis());
            user.setProfilePicture("img/favicon.png");
            userService.save(user);
        } catch (Exception e) {
            logger.error("create user failed, ", e);
            return result(ExceptionMsg.FAILED);
        }
        return new Response();
    }
    
    protected Response result(ExceptionMsg msg){
        return new Response(msg);
    }
    
}
```

注：后端返回Response的json数据。

6）创建service层和dao层类

a.创建UserService接口

```
package com.favorites.favoriteswebdemo.service;

import com.favorites.favoriteswebdemo.domain.User;

public interface UserService {
    public User findByEmail(String email);
    
    public User findByUserName(String userName);
    
    public User save(User user);
}
```

b.创建UserServiceImpl实现类

```
package com.favorites.favoriteswebdemo.service;

import javax.annotation.Resource;
import javax.transaction.Transactional;

import org.springframework.stereotype.Service;

import com.favorites.favoriteswebdemo.dao.UserRepository;
import com.favorites.favoriteswebdemo.domain.User;

@Service
@Transactional
public class UserServiceImpl implements UserService{
    
    @Resource
    private UserRepository userRepository;

    @Override
    public User findByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    @Override
    public User findByUserName(String userName) {
        return userRepository.findByUserName(userName);
    }

    @Override
    public User save(User user) {
        return userRepository.save(user);
    }

}
```

c.创建持久化类UserRepository

```
package com.favorites.favoriteswebdemo.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.favorites.favoriteswebdemo.domain.User;

public interface UserRepository extends JpaRepository<User, Long>{
    public User findByEmail(String email);
    
    public User findByUserName(String userName);
}
```