---
title: 页面提交数据库存储空字符串
tags: [work]
---

问题描述：

数据库使用mysql，jsp页面提交表单数据，当表单数据不填写时，存储到数据库中的字段为空字符串，而不是null。造成的问题是，可能会出现唯一键问题。

如果将某个字段值设置为唯一键，而当存储空字符串时，就会出现重复值错误。

错误排除：

数据库设置的默认值为NULL

![](/images/work/mysql/field-emptyvalue.png)

### 3种解决方式

方式1：js提交表单时，空字符串不提交

方式2：spring mvc封装对象时，处理空字符串为null。

参考：http://www.cnblogs.com/morethink/p/6435736.html

方式3：在程序中单独判断设置。