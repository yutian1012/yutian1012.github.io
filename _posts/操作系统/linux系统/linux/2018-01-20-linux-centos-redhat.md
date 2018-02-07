---
title: centos与redhat的关系
tags: [linux]
---

参考：http://blog.csdn.net/zx6268476/article/details/50709205

CentOS和RedHat虽然是两个独立的发行版，但是其实质是一样的。

1）GPL公开协议

GPL就是Linux内核所采用的软件许可证，GPL的特点是，你拿人家的代码修改用了，必须把修改后的代码公布。

所有的Linux都是采用的GPL许可，GPL许可允许GPL软件卖钱，但必须公布源码，所以每个Linux发行版的代码都是全公开的，只是，使用这些代码的人必须也公开修改过的代码。

2）Centos

CentOS（Community Enterprise Operating System）：RedHat的另一个重要分支，以RedHat所发布的源代码重建符合GPL许可协议的Linux系统。

由于Redhat的源代码是公开的，所以CentOS项目的人拿来自己再编译，同样的代码，同样的编译器，编译出来的自然是同样的东西。只不过里面删除了Redhat的Logo以及相应信息，而核心的管理工具还是rpm，只是用一个免费的软件包管理器yum（yellow dogupdate manager）替代了Redhat中的up2date，up2date更新是连接到Redhat的收费服务站点的，通过钱买来的服务代码通过认。

注：目前CentOS已被RedHat公司收购，但仍开源免费。

3）总结

RedHat在发行的时候都会同时提供二进制代码和源代码，无论是哪一种方式都可以免费从网络上获得，而CentOS所做的就是将RedHat发行的源代码重新编译，形成一个可用的二进制版本。CentOS在重新编译的时候不但保留了RedHat所有的功能，同时还做了不少功能上的优化。