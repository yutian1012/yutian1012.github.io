---
title: redis应用（二）
tags: [redis]
---

### 热点页面功能

页面显示最新注册的用户信息和最新的50条微博记录

1）显示微博热点数据页面timeline.php

由于要显示10位最新的注册用户，可在redis中构建一个链表，该链表维护10个最新的用户（存储userId）。每次注册成功都向该链表中添加数据。而在该页面显示最新用户时，可以通过该链表对象获取userId的集合

```
mv timeline.html timeline.php

<?php
    ...
    //从redis中获取最新注册用户链表（newuserlink）
    $r=connredis();
    //数组参数中的get相当于mysql中的连接，获取user表的username信息
    //user:userid:*中的*号
    $newuserlist=$r->sort('newuserlink',
        array('sore'=>'desc','get'=>'user:userid:*:username'))

?>

<div>
    <!-- 最新用户信息 -->
    <?php
        foreach($newuserlist as $v){
    ?>
        <a href="profile.php?u=<?php echo $v ?>"><?php echo $v ?></a>
    <?php
        }
    ?>
</div>

```

### 用户个人中心实现关注

具体步骤：获取用户；查询用户id是否在我的following（关注了谁）集合中；

1）当前的操作环境是：我登录了系统，来看别人的主页面。

```
mv profile.html profile.php

<?php
    ...
    //获取关注的用户
    $u=g('u');
    $uid=$r->get('user:username:'.$u.':userid');

    if(!$uid){
        error("用户不存在");
        exit;
    }

    $r=connredis();
    //获取当前用户已关注的集合，并且比较集合中是否已经包含了将要关注的用户
    $isf=$r->ismember('following:'.$user['userid'],$uid);
    $isfstatus=$isf?'0':'1';
    $isfword=$isf?'取消关注':'关注';

?>

<!-- 关注连接 -->
<a href="follow.php?uid=<?php echo $uid ?>&f=<?php echo $$isfstatus; ?>"><?php echo $isfword; ?></a>
...
```

2）实现关注功能

奖关系保持到redis数据库中，每个人都有自己的粉丝记录（被谁关注了）使用一个set集合来记录（follower）。同样，每个人也有关注的记录（关注了谁）也是有一个set集合来记录(following)。

注：关注了一个人需要向关注集合中添加元素，同时被关注的用户也要向粉丝集合中记录数据，是两个方向的操作。

具体步骤：判断uid，f是否合法；uid是否是自己；保存数据到关注集合和对方的被被关注集合

```
vim follow.php

<?php
    ...
    $uid=g("uid");//关注的用户id
    $f=g("f");//关注状态

    $r=connredis();
    //获取被关注的用户信息
    $uname=$r->get('user:userid:'.$uid.':username');
    
    if($f==1){//添加关注
        $r->sadd('following:'.$user['userid'],$uid);
        $r->sadd('follower:'.$uid,$user['userid']);
    }else{//取消关注
        $r->srem('following:'.$user['userid'],$uid);
        $r->srem('follower:'.$uid,$user['userid']);
    }

    header('Location: profile.php?u='.$uname);
?>
```

### 展示关注的微博信息

用户的home页面要展示关注人的微博信息。

实现方式有两种：

方式一，在被关注的对象写微博时，直接将数据推送给所有关注了该对象的人（follower被关注对象集合）。将用户查看微博信息保存到单独的一个结构体中，默认只保存100条，多余的数据保存到mysql数据库中。

```
# 使用集合保存微博数据
receivepost:fans:fansid
```

方式二，被关注的用户登录微博，访问home页面时主动去拉取数据。遍历所有的关注对象（following关注的对象集合），通过集合中的userid获取该用户发送的最新微博数据。

1）使用推送的方式实现

发送微博时，同时将微博信息推送给自己的粉丝。将用户查看微博信息保存到单独的一个结构体中，默认只保存100条，多余的数据保存到mysql数据库中。

```
vim post.php

<?php
    ...
    //获取粉丝信息
    $fans=$r->smembers('follower:'.$user['userid']);

    //将微博信息推送给粉丝
    $fans[]=$user['userid'];
    foreach($fans as $fansid){
        $r->lpush('receivepost:'.$fansid,$postid);
    }
?>
```

home页面显示关注人的微博信息

```
vim home.php

<?php
    ...
    
    //取出接收到的微博信息
    //$r->ltrim('receivepost:'.$user['userid'],0,49);
    $newpost=$r->sort('receivepost:'.$user['userid'],array('sort'=>'desc','get'=>'post:postid:*:content'));
    
    //粉丝的个数
    $myfans=$r->sCard('follower:'.$user['userid']);
    $mystar=$r->SCard('following:'.$user['userid']);()
?>

<!--展示自己的微博信息-->
<?php
    foreach($newpost as $p){
?>
    <div>显示收到的微博信息...</div>
<?php      
    }
?>

<!-- 页面显示粉丝数量和关注数量-->
<?php echo $myfans; ?> 粉丝
<?php echo $mystar; ?> 关注
```