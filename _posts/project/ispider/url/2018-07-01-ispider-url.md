---
title: URL调度系统
tags: [ispider]
---

1）URL仓库

使用redis来保存待检索的url。url仓库中的url地址在获取时的策略是通过队列的方式来实现的。

url分为种子URL列表，高优先级URL队列和低优先级URL队列三种类型。

种子URL列表是有URL定时器使用，通过种子URL获取待爬取的url，放入到redis仓库中，供爬虫节点使用。高优先级url其实是列表页url，列表页包含多个商品列表以及分页数据（该地址不是一个具体的商品详情页面）通过对高优先级url的解析可以获取具体商品的url地址（即低优先级url）。因此，低优先级url其实就是具体的商品页面。

2）URL调度器

调度是指将url放入到仓库或从仓库中取出url。

注：放入url时，分为高优先级队列和低优先级队列，而取数据时，则优先获取高优先级队列中的url。

IRepository接口定义了url调用方法。实现类RandomRedisRepositoryImpl，该类的注释中详细介绍了url分类，及其关联的常量SpiderConstants类。

3）URL定时器

UrlJob每天定时从url仓库中获取种子url，添加进高优先级列表，是定时器的工作线程。

4）URL分析

