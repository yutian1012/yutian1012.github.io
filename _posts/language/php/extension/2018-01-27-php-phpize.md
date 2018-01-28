---
title: phpize文件
tags: [php]
---

参考：https://www.zhihu.com/question/52844058

phpize是一个shell脚本，用于生成PECL扩展的configure文件。编译PHP扩展的工具，主要是根据系统信息生成对应的configure文件。

1）作用

phpize是用来扩展php扩展模块的，通过phpize可以建立php的外挂模块。比如你想在原来编译好的php中加入memcached或者ImageMagick等扩展模块，可以使用phpize，通过以下几步工作。

###实战：将memcached扩展模块加入到php中。

1）phpize文件目录

当php编译完成后，php的bin目录下会有phpize这个脚本文件。在编译你要添加的扩展模块之前，执行以下phpize就可以了。

```
# phpize文件目录
${PHP-HOME}/bin/phpize
```

2）下载memcache扩展模块

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

3）php中加载扩展

```
cd ${PHP-HOME}/lib
# 编辑配置文件
vi php.ini
# 指定生成的memcached扩展文件（.so）的路径
extension_dir="xxx/xxx"

extension=memcache.so
```

4）启动php

```
${PHP-HOME}/sbin/php-fpm
```

5）测试是否成功加载了扩展

```
vi test.php

<?php
    phpinfo();
?>
```

注：如果配置了nginx服务，可以访问该test.php文件，查看输出中是否存在memcache。 

6）php编写memcache操作

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