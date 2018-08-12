---
title: jvm java虚拟机栈-锁
tags: [jvm]
---

参考：http://www.cnblogs.com/kongzhongqijing/articles/3630264.html

Monitor是Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者Class的锁。每一个对象都有，也仅有一个monitor。