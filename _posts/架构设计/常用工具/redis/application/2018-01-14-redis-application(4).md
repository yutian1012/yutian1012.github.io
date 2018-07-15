---
title: redis应用（四）
tags: [redis]
---

### 字段设计过多的问题

1）问题：

微博信息在展示到页面中时，需要获取发微博人，发微博时间以及微博内容。最初设计时，微博信息获取是分了三个字段，获取时就需要获取3次，效率太低。

```
# 微博信息的保存结构
incr global:postid
post:postid:3:time timestamp
post:postid:3:userid 5
post:postid:3:content 'this is my home'
```

2）解决方案（使用hash结构）

可以将微博信息封装到hash结构中，获取数据时只需要获取hash值即可将全部的微博信息获取到了。

```
# 存储方式
post:postid:3 hash数据
# hash结构
userid=>用户id
time=>发送时间
content=>发送内容
```

3）代码实现

hash方式保存微博信息

```
vim post.php

<?php
    ...
    //保存微博信息
    $r->hmset('post:postid:'.$postid,
        array('userid'=>$user['userid'],'time'=>time(),'content'=>$content));

    //向粉丝中推送数据
    ...
?>
```

获取显示微博内容

```
<?php
    ...
    //获取的是postid集合信息
    $newpost=$r->sort('recivepost:'.$user['userid'],array('sort'=>'desc'));
?>

<!--显示微博信息-->
<?php
    foreach($newpost as $postid){
        $p=$r->hmget('post:postid:'.$postid,array('userid','time','content'));
?>
    <div class='post'>
        <a href='profile.php?'><?php echo $p['userid']; ?></a>
        <?php echo $p['content']; ?><br/>
        <i><?php echo $p['time'];?></i>
    </div>
<?php
    }
?>
```