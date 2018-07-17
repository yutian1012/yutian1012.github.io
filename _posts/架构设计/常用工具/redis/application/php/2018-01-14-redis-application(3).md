---
title: redis应用（三）
tags: [redis]
---

### cookie的安全性

cookie本身的数据是很容易修改，在火狐浏览器中就可以通过F12功能修改cookie的值。

解决方法：向cookie中添加salt，在登录时添加一个随机数，该随机数是与用户绑定的。cookie判断时会根据该随机数与用户对应关系判断。

1）编辑登录功能

```
vim login.php

<?php
    ...

    //判断用户是否已经登录
    function isLogin(){
        if(!$_COOKIE['userid'] || !$_COOKIE['username']){
            return false;
        }
        if(!$_COOKIE['authsecret']){
            return false;
        }else{
            $authsecret=$r->get('user:userid:'.$_COOKIE['userid'].':authsecret');
            if($authsecret!=$_COOKIE['authsecret']){
                return false;
            }
        }

        return array('userid'=>$_COOKIE['userid'],
            'username'=>$_COOKIE['username']);

    }

    ...
    $authsecret=randsecret();
    setcookie('authsecret',$authsecret);
    //关联用户与随机密码
    $r->set('user:userid:'.$userid.':authsecret',$authsecret);

?>
```

2）向公共库中添加一个参数随机数的函数

```
vim lib.php

function randsecret(){
    $str='abcdefghijklmnopqrstuvwxyz1234567890)(**&^%$#@!';
    $str=str_shuffle($str);//打乱字符串的顺序
    return sub_string($str,0,16);
}
```

3）退出时清空随机授权码

退出时除了要清空cookie信息，同时也需要清空redis中存放的随机授权码。

```
vim logout.php

<?php
    //清空cookie
    ...
    //清空redis中管理的随机授权码
    $r->set('user:userid:'.$userid.'authsecret','');
?>
```