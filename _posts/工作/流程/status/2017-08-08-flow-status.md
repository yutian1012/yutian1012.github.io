---
title: 流程状态信息
tags: [bpmx]
---

1）流程状态字段定义在ProcessRun类中

```
/** 挂起状态 */
public static final Short STATUS_SUSPEND = 0;
/** 运行状态 */
public static final Short STATUS_RUNNING = 1;
/** 结束状态 */
public static final Short STATUS_FINISH = 2;
/** 人工终止 */
public static final Short STATUS_MANUAL_FINISH = 3;
/** 草稿状态 */
public static final Short STATUS_FORM=4;
/** 已撤销 */
public static final Short STATUS_RECOVER = 5;
/** 已驳回 */
public static final Short STATUS_REJECT = 6;
/** 已追回 */
public static final Short STATUS_REDO = 7;
/** 逻辑删除 */
public static final Short STATUS_DELETE = 10;
```

2）流程状态标签类ProcessStatusTag

```
private String getProcessStatus(Short status) {
    switch (status) {   
        case 0:
            return "<font color='red'>挂起</font>";
        case 1:
            return "<font color='green'>审批中</font>";
        case 2:
            return "<font color='blue'>已结束</font>";
        case 3:
            return "<font color='red'>人工结束</font>";
        case 4:
            return "<font color='brown'>草稿</font>";
        case 5:
            return "<font color='red'>撤销</font>";
        case 6:
            return "<font color='red'>驳回</font>";
        case 7:
            return "<font color='red'>追回</font>";
        case 10:
            return "<font color='red'>逻辑删除</font>";
        default:
            break;
    }   
    return "";
}
```

3）使用

查看个人办公--我的请求

```
<f:processStatus status="${processRunItem.status}"></f:processStatus>
```

在bpm_pro_run_his和bpm_pro_run表的字段status中存放着流程的案件状态