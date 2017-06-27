---
title: linux开机启动设置
tags: [linux]
---

参考：http://www.jb51.net/article/74603.htm

方式一：图像界面方式设置

ntsysv

/etc/rc.d/* 中的程序一般就在/etc/init.d/*内, 往往是一些软链接(本人的debian 中没有/etc/rc.d目录)
/etc/init.d/中往往是一些开机启动要用的程序和服务. 不同的/etc/rcX.d(X=0,1,2,3,...), 对应不同的运行级别, 以不同的级别运行时, 会选用不同的/etc/rcX.d中的程序, /etc/rcX.d里的程序都是/etc/init.d中程序的软链接.
功能用法上没有什么可比性. /etc/init.d中就是一些程序, /etc/rcX.d则是规定了以何种方式使用这些程序