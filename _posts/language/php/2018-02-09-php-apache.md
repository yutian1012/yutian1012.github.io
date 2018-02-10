---
title: apache整合php
tags: [php]
---

1）在Apache的httpd.conf文件中添加php模块

```
# 在php的LoadModule模块中添加php的配置信息

# php7 support
# 让Apache加载php处理模块
LoadModule php7_module E:/work/installer/php-7.0.27/php7apache2_4.dll
# 该配置表示，当有一个资源是*.php的时候就由php来处理
# 后面的.html .htm都可以交由php来处理
AddType application/x-httpd-php .php .html .htm
# 在php的解压目录下存在php.ini文件，因此这里配置php的解压目录即可
PHPIniDir "E:/work/installer/php-7.0.27/"
```

注：php配置文件，把php.ini-development复制一份，改变后缀php.ini，因为php的设置需要在php.ini修改。

2）让apache把index.php也设为默认页

```
# 在IfModule中将index.php加入配置
<IfModule dir_module>
    DirectoryIndex index.html index.php
</IfModule>
```

3）检查配置是否成功并启动

```
# 检查配置文件
httpd -t
# 重新启动apache
httpd -k restart
```

4）新建php测试页面，并访问

在htdocs文件夹下创建hello.php

```
<?php
    phpinfo();
?>

# 访问php页面
http://localhost/hello.php
```

注：如果正确输出了php的配置相关信息，那么这里php的运行环境就已经搭建成功了。