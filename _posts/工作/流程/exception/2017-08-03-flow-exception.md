---
title: 流程后置处理器中抛出异常
tags: [bpmx]
---

1）场景：

在流程的后置处理器中处理用户表单的后台校验，校验不成功抛出异常。而任务已经显示执行完成了，没有实现回滚操作。另外，在下一个节点执行回退操作，出现错误。原因是没有向executionStackService中存放节点执行的信息。即抛出异常后没有向堆栈中放置执行信息，造成回退操作时从堆栈获取上一步的执行操作出现空指针问题。

参考：http://blog.csdn.net/yipanbo/article/details/46048413

问题1：后置处理抛出异常后，任务没有实现回滚。

原因是抛出的异常时Exception，spring aop不能回滚Exception异常。

方式一：catch块中手动抛出RuntimeException

抛出的异常需要使用RuntimeException，才能实现回滚。在捕获异常后重新将异常包装成RuntimeException后再抛出。

```
try {          
    userDao.save(user);          
   } catch (Exception e) {         
    logger.info("异常信息："+e);          
    throw new RuntimeException();         
}
```

注意：在使用第三方框架时，有时可能出现对异常进行包装的情况，这个时候要小心RuntimeException被包装成其他类型的异常，导致事务无法回滚。

方式二：catch框中设置事务回滚

通过TransactionAspectSupport手动设置事务回滚

```
try {          
    userDao.save(user);          
   } catch (Exception e) {         
    logger.info("异常信息："+e);          
    TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();     
 }
```

2）总结

spring aop异常捕获原理：被拦截的方法需显式抛出异常，并不能经任何处理，这样aop代理才能捕获到方法的异常，才能进行回滚，默认情况下aop只捕获runtimeexception的异常。

问题2：后置处理抛出异常后，没有向执行堆栈中设置执行信息，造成回退操作空指针异常。
