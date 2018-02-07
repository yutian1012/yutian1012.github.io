---
title: 任务评论
tags: [activiti]
---

流程执行时，针对任务节点提交人可能会提交自己的意见，activiti可以将任务或流程的评论（意见）信息保存到ACT_HI_COMMENT表中。对应的实体类为CommentEntity。

1）实体类CommentEntity

```
protected String id;
protected String type;//数据类型，对应TYPE_字段，值为event或comment
protected String userId;//提交评论的userId
protected Date time;//评论时间
protected String taskId;//评论数据对应的任务ID
protected String processInstanceId;//评论数据对应的流程ID
protected String action;数据的操作标识
protected String message;//评论的数据信息
protected String fullMessage;//评论的数据信息
```

注：message与fullMessage的区别，message存储字符串数据，fullMessage存储对应的byte字节内容。

2）新增任务评论

```
taskService.addComment(task.getId(), pi.getId(), "this is comment message");
```

![](/images/book/workflow/activiti/task/comment/addcomment.png)

注：其中TYPE_字段值为comment，ACTION_字段值为AddComment

3）事件的记录

针对taskServiced 操作都会记录event事件

```
// 调用各个记录事件的方法 
taskService.addComment(task.getId(), pi.getId(), "this is comment message");
taskService.addUserIdentityLink(task.getId(), "1", "user");
taskService.deleteUserIdentityLink(task.getId(), "1", "user");
taskService.addGroupIdentityLink(task.getId(), "1", "group");
taskService.deleteGroupIdentityLink(task.getId(), "1", "group");
Attachment atta = taskService.createAttachment("test", task.getId(), pi.getId(), "test", "test", "");
taskService.deleteAttachment(atta.getId());

// 查询Comment和Event
List<Comment> comments = taskService.getTaskComments(task.getId());
System.out.println("总共的评论数量：" + comments.size());//1
List<Event> events = taskService.getTaskEvents(task.getId());
System.out.println("总共的事件数量：" + events.size());//7
```

注：taskService.getTaskEvents方法也能获取addComment方法添加的评论，尽管数据的TYPE_字段值为comment（也被当作事件来处理了）。

注2：历史数据配置（Activiti配置文件processEngineConfiguration配置中的history属性），必须不能为none。如果将history设置为none，那么将不会进行事件记录，并且在调用addComment方法时，将会抛出异常：history is not enabled

4）评论数据查询

TaskService提供了根据tskId查询评论，查询事件，以及根据流程ID查询评论/事件方法。

```
// 调用各个记录事件的方法 
taskService.addComment(task.getId(), pi.getId(), "this is comment message"); 
taskService.addUserIdentityLink(task.getId(), "1", "user");
taskService.deleteUserIdentityLink(task.getId(), "1", "user");
taskService.addGroupIdentityLink(task.getId(), "1", "group");
taskService.deleteGroupIdentityLink(task.getId(), "1", "group");
Attachment atta = taskService.createAttachment("test", task.getId(), pi.getId(), "test", "test", "");
taskService.deleteAttachment(atta.getId());
// 查询事件与评论
List<Comment> commonts1 = taskService.getProcessInstanceComments(pi.getId());
System.out.println("流程评论（事件）数量：" + commonts1.size());//2
commonts1 = taskService.getTaskComments(task.getId());
System.out.println("任务评论数量：" + commonts1.size());//1
List<Event> events = taskService.getTaskEvents(task.getId());
System.out.println("事件数量：" + events.size());//7
```

注：getProcessInstanceComments会获取addComent添加的评论，以及createAttachment添加的附件事件。该方法不会判断数据类型是否为comment，只要流程实例id匹配就会将数据查询出来。
