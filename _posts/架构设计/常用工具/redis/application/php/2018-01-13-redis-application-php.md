---
title: redis应用在php中
tags: [redis]
---

使用php语言开发项目，然后php要访问redis数据库，需要提供相应的扩展。

### php-redis的扩展编译

1）下载

到http://pecl.php.net/网站上搜索redis，下载文档版的扩展。

2）安装扩展

```
# 下载扩展源码包
cd /usr/local/src
wget http://pecl.php.net/get/redis-2.2.5.tgz

# 解压
tar zxvf redis-2.2.5.tgz
# 该目录下只有一些源代码，甚至config问题都没有
# 具体config适合php版本相关的，所以要根据php的版本来生成config
cd redis-2.2.5.tgz

# 在源码目录下，使用php安装目录下的工具phpize动态执行，生成config文件
/usr/local/fastphp/bin/phpize
# 执行完成命令后，会出现configure文件，执行该configure命令
./configure --with-php-config=/usr/local/fastphp/bin/php-config

# 安装，会将生成的so文件放在
# /usr/local/fastphp/lib/php/extensions/no-debug-non-zts-20100525/.redis.so
make && make install

# php中引入redis.so插件
# 编辑php.ini文件

cd /usr/local/fastphp/lib
vim php.ini

# 添加扩展
extension=/usr/local/fastphp/lib/php/extensions/no-debug-non-zts-20100525/.redis.so

# 重启php
pkill -9 php-fpm
/usr/local/fastphp/sbin/php-fpm
```

3）redis插件的使用

创建php页面测试php连接并使用redis

```
cd /usr/local/nginx/html
vim redis.php

<?php
    
    # 创建对象
    $redis = new Redis();

    // connect to redis server
    $redis->open('localhost',6379);
    //设置数据
    $redis->set('user:id:1:name','wangwu');
    # 获取并输出
    var_dump($redis->get('user:id:1:name'));
?>
```