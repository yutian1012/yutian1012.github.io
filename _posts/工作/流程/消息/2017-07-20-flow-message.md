---
title: 流程中的消息提醒
tags: [bpmx]
---

1）启动流程：

ProcessRunService.java类中的startProcess方法中会调用消息发送

2）消息发送

```
taskMessageService.notify(TaskThreadService.getNewTasks(), "", processRun.getSubject(), null, "", "");
```

调用notify方法通知候选人审批信息，调用taskMessageService类的sendMsg方法

```
sendMsg(task, map, user, informTypes, parentActDefId, bpmNodeSet, subject, opinion);
```

该方法会将信息封装到MessgeModel中，并借助消息中间件发送。然后交由相应的监听处理类进一步完成。

```
MessageModel messageModel = new MessageModel(infoType);
messageModel.setReceiveUser((ISysUser) user);
messageModel.setSendUser( curUser);
...
messageModel.setIsTask(true);
messageModel.setOpinion(opinion);
messageProducer.send(messageModel);
```
