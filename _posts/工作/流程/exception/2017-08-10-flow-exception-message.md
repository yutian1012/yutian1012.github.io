---
title: 异常信息的获取
tags: [bpmx]
---

在流程后置处理器中如何获取抛出的异常信息。

1）流程前/后置处理器的处理

```
private void invokeHandler(ProcessCmd processCmd, BpmNodeSet bpmNodeSet, boolean isBefore) throws Exception {
    if (bpmNodeSet == null)
        return;
    String handler = "";
    if (isBefore) {
        handler = bpmNodeSet.getBeforeHandler();
    } else {
        handler = bpmNodeSet.getAfterHandler();
    }
    if (StringUtil.isEmpty(handler))
        return;

    String[] aryHandler = handler.split("[.]");
    if (aryHandler != null) {
        String beanId = aryHandler[0];
        String method = aryHandler[1];
        // 触发该Bean下的业务方法
        Object serviceBean = AppUtil.getBean(beanId);
        if (serviceBean != null) {
            Method invokeMethod = serviceBean.getClass().getDeclaredMethod(method, new Class[] { ProcessCmd.class });
            invokeMethod.invoke(serviceBean, processCmd);
        }
    }
}
```

注：实际是使用Method的invoke调用的，如果方法内部抛出Exception，那么异常信息会被包装成InvocationTargetException。

2）打印出异常信息

```
try {// TODO
    String[] aryHandler = handler.split("[.]");
    if (aryHandler != null) {
        String beanId = aryHandler[0];
        String method = aryHandler[1];
        // 触发该Bean下的业务方法
        Object serviceBean = AppUtil.getBean(beanId);
        if (serviceBean != null) {
            Method invokeMethod = serviceBean.getClass().getDeclaredMethod(method, new Class[] { ProcessCmd.class });
            invokeMethod.invoke(serviceBean, processCmd);
        }
    }
} catch (Exception e) {
    //异常类型被包装java.lang.reflect.InvocationTargetException
    //System.out.println(e.getClass());
    //System.out.println(e.getMessage());//输出空
    //输出调用方法中实际抛出的异常内容
    //System.out.println(e.getCause().getMessage());
    //再次包装成RuntimeException，为了事务回滚做准备
    throw new RuntimeException(e.getCause().getMessage());
}
```

注：获取包装的内部异常信息使用e.getCause().getMessage()方法。