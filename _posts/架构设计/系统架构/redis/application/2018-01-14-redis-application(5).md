---
title: redis应用（五）--拉模式
tags: [redis]
---

### 推送模式存在的问题

1）关注用户的限制

推送的缺点，有些用户被很多人所关注，如有5000万用户关注。如果该用户发送一条微博信息，那么数据库就要存放5000万条冗余的微博主键数据，操作非常耗时。

解决方法：可以考虑使用拉模式，当用户登录后，主动去查询用户所关注人员所发生的微博信息。

注：在新浪微博中关注人最多只允许关注1000左右的人，从而也限定了查询数据的量级。

2）数据显示的限制

另外，拉取出的数据也并不会全部显示，如关注的用户发了大概1万条微博数据，这些数据都显示吗？显然不是，只会显示热点最新的数据。

注：在新浪微博中也只会显示出1000条数据左右，并不会全部显示。

### 拉取模式问题

1）问题，如何拉取：

上次拉取了A用户的5，6，7三条微博，下次刷新home.php，应该从大于7的微博开始拉取。这里必须要记录相应的开始拉去微博的最大键值，或者维护一个拉取的时间点，通过时间点来比较获取新的数据。

解决方法：

不采用：每次来取时，设置一个lastpull时间点，下次拉取时只取出时间大于lastpull的微博。

因为时间点存储在微博的hash结构中，不方便比较，因此可以通过微博的主键进行比较。每次取微博时，设置一个最大的微博id字段，用来存储取出的最大的微博主键，下次同步时，使用该键值在去拉取。

2）问题，如何拉取关注集合的微博信息

针对关注的人集合的处理，循环自己的关注列表，for遍历的方式逐个取出他们的微博信息。

3）问题，取出的数据放置到什么地方？

可以设置一个链表（pull:$userid）用于存放拉取的数据，即将取出的信息放置到链表中。

4）问题，如果保证个人中心只有前1000条数据

使用ltrim支取1000条，将多余的数据清空。

5）问题，多个关注用户的记录如何显示（排序问题）

取出所有的微博数据后，需要将微博数据按照时间进行排序。发布数据时发布的信息使用的是hash结构，微博的主键，发微博的时间等都存放在hash结构中，而hash结构的数据是不方便进行排序的。

解决：不通过时间，而是通过postid（微博的主键）进行排序，因为微博的主键字段是全局递增的。因此按照微博主键降序即可实现按时间排序。



### 拉取模式的设计与实现

1）拉取数据表的key设计

```
# 保存用户获取的最大的postid值
lastpull:userid:$userid 
# 用户发送的微博集合，保存20条记录数据，供用户拉取时查找粉丝发的微博信息
starpost:userid:$userid
```

注1：与推数据的比较中需要增加这两个键值，推送模式是直接向receivepost:$userid中推入lpush数据，用户显示微博信息时，直接从receivepost:$userid集合中获取数据展示即可。

注2：拉模式稍微复杂一点，需要遍历所有的关注对象，从关注对象中获取发送微博信息集合（starpost:userid:$userid），并且在获取时要比较已经获取的数据的最大postid，防止获取重复数据。将获取的数据放置到receivepost:$userid集合中，用户显示微博信息时，也是从receivepost:$userid集合中获取数据的。

2）实现代码

```
vim home.php

<?php
    ...
    //获取已经取到的最新的微博主键
    $lastpull =$->get('lastpull:userid:'.$user['userid']);
    if(!$lastpull){
        $lastpull=0;
    }
    //主动取出关注对象发生的微博信息
    $star=$r->smembers('following:'.$user['userid']);
    $star[]=$user['userid'];//将自己的id也放入数组中
    //读取最新数据，将数据读取到一个集合中
    $lastest=array();
    foreach($star as $s){
        $maxpostid=$r->get('global:postid');
        //合并数组的值
        $lastest=array_merge($lastest,
            $r->zrangebyscore('starpost:userid:'.$s,$lastpull+1,$maxpostid));
    }
    //对获取的集合元素进行排序，使用php的排序方式
    sort($lastes,SORT_NUMBERIC);
    //更新lastpull字段
    if(!empty($lastest)){
        $r->set('lastpull:userid:'.$user['userid'],end($lastes));    
    }

    //将集合数据设置到receivepost中，将信息展示到页面上
    foreach($lastest as $l){
        $r->lpush('receivepost:'.$user['userid'],$l);
    }
    //保持个人主页最多1000条数据
    $r->ltrim('receivepost:'.$user['userid']，0，999);
?>
```

修改post的发送数据，发送时将用户的发送的postid也记录到一个集合中

```
vim post.php

<?php
    ...
    //发送微博信息
    $r->hmset('post:postid:'.$postid,
        array('userid'=>$user['userid'],'time'=>time(),'content'=>$content));

    //将自己发的微博维护在一个集合里，第一个$postid作为排序的score
    $r->zadd('starpost:userid:'.$user['userid'],$postid,$postid);
    //只需要前20条数据即可，并不需要记录太多，保证最新的20条数据
    if($r->zCard('starpost:userid:'.$user['userid'])>20){
        $r->zRemRangeByRank('starpost:userid:'.$user['userid'],0,0);
    }
    $r->ltrim('startpost:userid:'.$user['userid'],0,19);
?>
```