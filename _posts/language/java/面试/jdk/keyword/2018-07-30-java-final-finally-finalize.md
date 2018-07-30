---
title: java区分final、finally和finalize
tags: [interview]
---

1）final

final用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，类不可继承。

注意：如果是基本类型说明变量本身不能改变，如果是引用类型，说明它不能指向其他的对象了。但对象还是可以改变的。

2）finally

finally是异常处理语句结构的一部分，表示无论是否出现异常总是执行。

注：与return的执行顺序，可参考java-finally文章

3）finalize

finalize是Object类的一个方法，可以为任何一个类添加finalize方法，在垃圾收集器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源回收。

注：finalize()的调用具有不确定行，只保证方法会调用，但不保证方法里的任务会被执行完。