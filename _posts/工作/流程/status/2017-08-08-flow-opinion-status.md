---
title: 流程中的意见数值
tags: [bpmx]
---

taskOpinion类中存在很多的int常量值，这些值有什么意义？

```
public final static Short STATUS_INIT = (short) -2;
/**
 * 正在审批
 */
public final static Short STATUS_CHECKING = (short) -1;
/**
 * 弃权=0
 */
public final static Short STATUS_ABANDON = (short) 0;
/**
 * 同意=1
 */
public final static Short STATUS_AGREE = (short) 1;
...
```