---
title: redis应用（六）--与mysql结合
tags: [redis]
---

每个人的微博数据在redis中最大存储1000条，将多余的数据保存到mysql中，需要检索时再从数据库中获取。

### 开发思路：

1）判断并转移微博数据到global链表

在用户发送微博时，判断用户发送的微博数量，如果超过1000条微博（从用户发送的微博集合中，redis维护的集合变量），则将集合中的最长时间的微博转移到一个global的链表中

2）向redis中写入新的微博数据

向用户发送微博的集合中写入新的微博数据。

3）定时计划将旧的微博数据写入到mysql

系统运行一个定时器，定时从global链表中获取微博数据转存到mysql数据库中。

4）涉及到的数据结构

```
# 记录自己发送的postid
mypost:userid:$userid
# 全局的集合，用来存放旧的微博数据，定时器访问该集合，将数据转存mysql
global:store
```

### 开发问题：

1）用户如何操作自己发送的微博数据？

发送微博时维护一个集合，该集合中存放用户发送的微博，集合最大存储1000条微博数据，多余的微博存放到mysql中。

创建一个redis的集合，便于用户使用自己的id查看已经发送的微博信息

```
# 记录自己发送的postid
mypost:userid:$userid
```

### 代码实现

1）改造发送微博的代码

```
vim post.php

<?php
    ...
    //把自己的微博id放到一个链表中，链表大小为1000，使用自己的userid做索引
    $r->lpush('mypost:userid:'.$user['userid'],$postid);
    //判断是否超出1000条数据
    if($r->llen('mypost:userid:'.$user['userid'])>1000){
        $r->rpoplpush('mypost:userid:'.$user['userid'],'global:store');
    }
?>
```

2）创建mysql数据表

```
create table post(
    postid bigint primary key,
    userid int ,
    username varchar(20) not null default '';
    time int not null default 0,
    content varchar(144)
)engine myisam charset utf8;
```

3）定时器保存数据到mysql中

```
vim cron2mysql.php

<?php
    include('./lib.php');

    //连接数据库
    $conn=mysql_connect('127.0.0.1','root','');
    mysql_query('user test',$conn);
    mysql_query('select names utf8',$conn);//设置字符集

    $sql='insert into post(postid,userid,username,time,content) values'
    //连接redis
    $r=connreds();
    //从全局集合中获取待转存到mysql的微博数据，每次循环最大取1000条数据
    $i=0;
    while($r->llen('global:store')&& $i++<1000){
        $postid=$r->rpop('global:store');
        //获取微博信息
        $post=$r->hmget('post:postid:'.$postid,
            array('userid','username','time','content'));
        //拼接sql，针对字符串需要使用单引号括起来
        $sql.="(postid,". $post['userid'] . "'," . $post['username'] . "," . $post['time'] . ",'" . $post['content'] ."'"),;
    }

    $sql=substr($sql,0,-1);
    //执行sql
    mysql_query($sql,$conn);
?>
```

4）设置定时器

```
crontab -e

# 添加定时器
*/1 * * * * /usr/local/php/bin/php /usr/local/httpd/htdocs/weibo/cron2mysql.php
```

注：如果使用定时器执行，需要将cron2mysql.php中include的lib.php改成绝对路径，否则不能成功（shell环境变量问题）