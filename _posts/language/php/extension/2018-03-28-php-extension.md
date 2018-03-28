---
title: php扩展功能
tags: [php]
---

php加载扩展的方式

1）方式一：动态加载，指定目录

php配置文件php.ini中的extension_dir路径指定了扩展路径。php的扩展包一般会放置在该路径下。

```
# windows配置
extension_dir = "E:/work/installer/php-7.0.27/ext"
# linux常用配置
extension_dir = "/usr/local/php/lib/php/extensions"

# 当要加载某个包时，只需要填写文件名即可，不用再填写路径了
# msql.so放置到extension_dir目录下即可，两者结合就能找到扩展包了
extension=msql.so
```

注：将要加载的包拷贝到该目录下即可

2）方式二：指定特定的扩展包

使用extension直接指定要加载的包。

```
# 直接指定要扩展的软件包
extension=/path/to/extension/msql.so
```