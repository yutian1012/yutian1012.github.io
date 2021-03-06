---
title: 淘宝发布策略
tags: [architecture]
---

1）预发布和分批发布

预发布是这样的，比如说一个集群有200台机器，先把一台机器摘下来不提供服务，把程序推上去，再验证这个程序在这个地方是不是运行正常，如果运行正常那么预发布结束。然后从200台机器里先推50台，挂上去提供服务，如果运行正常，再把其他全部更新程序。在分批发布时如果运行出现异常必须保证退回到原状态。这是为了规避一些潜在的隐患，因为有些问题，在测试环境下不一定百分百覆盖到。

在预发布阶段，没有用户参与，测试手段就是自动化测试和内部脚本执行。先推的50台进去会对外提供服务，用户访问这50台机器，用户的流量就会进来。通过进来的流量后观察，包括系统监控，分析日志，查看错误日志等。

分批发布时为了缩小受用户的范围，因此上线的代码不能完全保证正确，为了规避风险，只能采取缩小影响用的范围。如果200台机器，第一次发布20台，也就是十分之一的用户受到影响。

2）灰度发布

一般情况下分批发布时之间隔比较短的发布，而灰度发布的话指间隔比较长，一般灰度要求至少24小时的间隔。就是说，如果第一批发布20台机器，要在一天内观察机器的运行情况，在确定是否继续发布还是回滚。

灰度实际上专业名词叫bucket test（就是桶检测）。抽一部分用户去检测。有几种方式，一个是按流量比例来抽，按流量来抽就是随机命中的方式。还有一种方式是根据用户来抽，如根据用户的cookie来判断是不是该进入到桶里来。

灰度发布的效率比较低，发布时要分地区分批，时间比较长，一般针对核心业务淘宝才会采取灰度发布。

注：一般采用的是预发布和分批发布。