---
title: 流程中的启动按钮事件
tags: [bpmx]
---

有时候在流程执行时，会根据不同的按钮触发不同操作。如同意执行下一步时，需要更新某个字段的值为1，而驳回时需要将其设置为0。因此，后台需要获取用户点击的按钮，从而判断数据如何更新。

1）在ProcessCmd对象中voteAgree字段

```
/**
 * 投票状态。
 * <pre>
 * 0=弃权， 1=同意
 * 2=反对， 3=驳回
 * 4=追回
 * 5=会签通过
 * 6=会签不通过
 * </pre>
 */
private Short voteAgree=1;
```

注：默认值为1，表示同意

2）在节点自定义按钮时选择为节点配置按钮

```
//开始按钮--开始节点只能配置下面按钮
/**
 * 提交
 */
public final static  int START_BUTTON_TYPE_START = 1;
/**
 * 流程示意图
 */
public final static  int START_BUTTON_TYPE_IMAGE = 2;
/**
 * 打印
 */
public final static  int START_BUTTON_TYPE_PRINT = 3;
/**
 * 保存草稿
 */
public final static  int START_BUTTON_TYPE_SAVE = 6;

//任务按钮--非起始节点配置按钮
/**
 * 同意
 */
public final static  int NODE_BUTTON_TYPE_COMPLETE = 1;
/**
 * 反对
 */
public final static  int NODE_BUTTON_TYPE_OPPOSE = 2;
/**
 * 弃权
 */
public final static  int NODE_BUTTON_TYPE_ABSTENT = 3;
/**
 * 驳回
 */
public final static  int NODE_BUTTON_TYPE_BACK = 4;
/**
 * 驳回到发起人
 */
public final static  int NODE_BUTTON_TYPE_BACKTOSTART = 5;
/**
 * 转办
 */
public final static  int NODE_BUTTON_TYPE_ASSIGNEE = 6;
/**
 * 补签
 */
public final static  int NODE_BUTTON_TYPE_ADDSIGN = 7;
/**
 * 保存表单
 */
public final static  int NODE_BUTTON_TYPE_SAVEFORM = 8;
/**
 * 流程示意图
 */
public final static  int NODE_BUTTON_TYPE_IMAGE = 9;
/**
 * 打印
 */
public final static  int NODE_BUTTON_TYPE_PRINT = 10;
/**
 * 审批历史
 */
public final static  int NODE_BUTTON_TYPE_HIS = 11;
/**
 * 终止
 */
public final static  int NODE_BUTTON_TYPE_END = 15;
/**
 * 沟通
 */
public final static  int NODE_BUTTON_TYPE_COMMU = 16;
/**
 * 转办管理员
 */
public final static  int NODE_BUTTON_TYPE_ASSIGNEEMANAGE = 17;
/**
 * 沟通反馈
 */
public final static  int NODE_BUTTON_TYPE_FEEDBACK = 18;
/**
 * 撤销
 */
public final static  int NODE_BUTTON_TYPE_REVERT = 43;
```

注：该按钮类型的int值对应了bpm_node_btn表的operatorType字段，即操作类型。

3）按钮button.xml文件