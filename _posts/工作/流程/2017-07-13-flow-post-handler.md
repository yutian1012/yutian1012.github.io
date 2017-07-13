---
title: 流程中的后置处理器
tags: [bpmx]
---

### 后置处理器定位

位置一：ProcessRunService类的nextProcess方法（流程执行下一步）

位置二：ProcessRunService类的startProcess方法（启动流程）

### 执行过程

1）执行判断

```
if (!processCmd.isSkipAfterHandler()) {
    invokeHandler(processCmd, bpmNodeSet, false);
}
```

注：会先判断流程设置中是否略过后置处理器的执行

2）调用ProcessRunService类的invokeHandler方法

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
        e.printStackTrace();
    }
}
```

注：通过代码可以看到，前置/后置处理器都是设置在bpmNodeSet对象中，对应表bpm_node_set表。通过获取设置值，从spring容器中取得业务类的实例，利用反射方式调用相应的方法。方法参数必须有且只有一个，并且类型为ProcessCmd。

注2：从获取spring容器bean实例的角度看，设置后置处理器时要根据bean在spring容器中的命名规范来设置。