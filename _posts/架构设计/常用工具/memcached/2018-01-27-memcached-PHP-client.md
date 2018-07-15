---
title: memcached的PHP客户端的使用
tags: [memecached]
---

php访问memcache是以扩展的方式安装的。

1）下载并安装memcache扩展模块

下载地址：http://pecl.php.net/package/memcached

```
# 将下载的软件放置到统一的目录下
cd /home/xxx/download
# 下载
wget http://pecl.php.net/get/memcached-3.0.0.tgz
# 解压并进入该目录
tar zxvf memcached-3.0.0.tgz
cd memcached-3.0.0
# 编译前执行
${PHP-HOME}/bin/phpize
# configure命令
./configure –with-php-config=${PHP-HOME}/bin/php-config
# 安装
make & make install
```

2）php中加载扩展

```
cd ${PHP-HOME}/lib
# 编辑配置文件
vi php.ini
# 指定生成的memcached扩展文件（.so）的路径
extension_dir="xxx/xxx"

extension=memcache.so
```

3）启动php

```
${PHP-HOME}/sbin/php-fpm
```

4）测试是否成功加载了扩展

```
vi test.php

<?php
    phpinfo();
?>
```

注：如果配置了nginx服务，可以访问该test.php文件，查看输出中是否存在memcache。 

5）php编写memcache操作

操作api手册：http://php.net/manual/zh/book.memcached.php

```
vi test.php

<?php
    $memcache=new Memcache;
    $memcache->connect('127.0.0.1',11211) or die("could not connect");

    $memcache->set('key','hello');
    $get_value=$memcache->get('key');
    echo $get_value;
?>
```