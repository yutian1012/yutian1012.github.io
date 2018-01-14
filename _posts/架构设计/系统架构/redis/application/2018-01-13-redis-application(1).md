---
title: redis应用（一）
tags: [redis]
---

将美工制作的页面模板放置到网站的根目录下，然后编辑页面。

### 页面的公共部分

1）页面头部

vim header.php，制作每个页面通用的head部分

```
# 将index.html制作的模板读入
:r index.html
# 只引入head的头数据
```

2）页面尾部

vim footer.php，制作每个页面通用的footer部分

3）index首页页面

```
mv index.html index.php

vim index.php

<?php include('head.php');?>

//首页内容信息

<?php include('footer.php');?>
```



### 注册用户

设计user表对应的key规则

1）注册用户

用户表只设计三个字段：userid，username和password，并且可以根据用户名获取用户信息。

```
set user:userid:1:username zhang
set user:userid:1:password 111111

# 提供能够根据用户名查找用户的功能
set user:username:zhang:userid 1
```

2）编辑注册用户页面

具体步骤为：

接收$_post参数，判断合法性；

连接redis，查询用户名，判断是否存在；

将用户信息写入redis；

完成登录操作；

```
vim register.php

<?php
    
    //引入head
    include('header.php');
    //引入公共封装库函数文件
    include('lib.php');
    
    $username=p('username');
    $password=p('password');
    $password2=p('password2')；

    //接收表单数据
    if(!$username)||!$password)||!$password2)){
        error('请输入正确的参数');
    }
    //判断密码是否一致
    if($password==$password2){
        error('请输入一致的密码');
    }
    //连接redis，判断用户名是否存在
    $r=connredis();
    //查询用户名是否已被注册
    if($r->get('user:username:'.$username.':userid')){
        error('用户名已被注册!');
    }

    //保存数据到数据库中
    //获取userId，通过redis中设置一个自增的global:userid，作为用户的主键字段值
    $userid=$r->incr('global:userid');

    $r->set('user:userid:'.$userid.':username',$username);
    $r->set('user:userid:'.$userid.':password',$password);
    //提供根据username获取用户信息
    $r->set('user:username:'.$username.':userid',$userid);

    //通过一个链表，维护10个最新用户，再热点页面会显示最新用户信息
    $r->lpush('newuserlink',$userid);
    //保证newuselink链表只有10个元素
    $r->ltrim('newuserlink',0,9);

    //引入footer信息
    include('footer.php');
?>
```

3）基础函数库

将公共的函数信息放置到该文件中，命名为lib.php

```
vim lib.php

<?php
    //获取页面传递的参数
    function p($key){
        return $_POST[$key];
    }
    //获取页面传递的参数
    function p($key){
        return $_GET[$key];
    }
    //错误信息提示函数
    function error($msg){
        echo $msg;
        include('./footer.php');
        exit;
    }
    //连接redis函数
    function connredis(){
        static $r=null;
        if($r!=null){
            return $r;
        }
        $r=new redis();
        $r->connect('localhost');
        return $r;
    }

    //判断用户是否登录
    function isLogin(){
        if(!$_COOKIE['userid'] || !$_COOKIE['username']){
            return false;
        }
        return array('userid'=>$_COOKIE['userid'],
            'username'=>$_COOKIE['username']);

    }
?>
```

### redis相关的操作

数据库中有时要初始化一些数据，用来辅助应用程序

```
# 提供一个全局的userid，为用户表提供一个自增的主键
set global:userid 1
```

### 登录功能

具体步骤：

接收$_POST，判断完整性；

查询用户名判断是否存在；

查询密码判断是否匹配；

设置cookie；

```
vim login.php

<?php
     //引入head
    include('header.php');
    //引入公共封装库函数文件
    include('lib.php');

    //判断是否登录。
    if(isLogin()!=false){
        header("location: home.php");
        exit;
    }

    $username=p('username');
    $password=p('password');

    if(!$username||!$password){
        error('请输入完整的用户名和密码');
    }

    $r=connredis();
    $useid=$r->get('user:username:'.$username.'userid');
    if(!$userid){
        error('用户名不存在');
    }

    $dbpassword=$r->get('user:userid:'.$userid.'password');
    if($password!==$dbpassword){
        error('密码输入错误');
    }

    //设置cookie，登录成功
    setcookie('username',$username);
    setcookie('userid',$userid);

    //登录成功跳转到home页面
    header('location:home.php');

    include('footer.php');
?>
```

### 主页home.php

```
mv home.html home.php

<?php
    //未登录则跳转到首页去登录
    if(($use=isLogin())==false){
        header('location: index.php');
    }
?>

//页面元素信息
<div>xxxx</div>
...
用户名：<?php echo $user['username']; ?>
...

//发送微博信息
<form>
    xxx
</form>

```

### 微博发送功能

1）微博数据存储

```
incr global:postid
post:postid:3:time timestamp
post:postid:3:userid 5
post:postid:3:content 'this is my home'

```

2）微博功能处理页面使用post.php

具体步骤：判断是否登录；接收post内容；向redis中保持微博内容；

```
vim post.php

<?php
    ...
    //判断是否登录。
    if(($user=isLogin())==false){
        header("location: index.php");
        exit;
    }

    $content=p('content');
    if(!$content){
        error('请填写内容');
    }

    $r =connredis();
    $postid=$->incr('global:postid');
    $r->set('post:postid:'.$postid.':userid',$user['userid']);
    $r->set('post:postid:'.$postid.':time',time());
    $r->set('post:postid:'.$postid.':content',$content);

    //显示用户发的所有微博信息？

    //你所关注的人员的微博信息？

    header('location: home.php');
?>
```

### 退出功能

```
vim logout.php

<?php
    setcookie('username','',-1);
    setcookie('userid','',-1);
    header('location: index.php');
?>
```