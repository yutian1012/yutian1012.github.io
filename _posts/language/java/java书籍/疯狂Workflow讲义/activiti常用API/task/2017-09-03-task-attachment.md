---
title: 任务附件管理
tags: [activiti]
---

许多流程中都会带上一些任务相关的附件。TaskService提供了对任务附件进行创建、删除和查询的操作。activiti用ACT_HI_ATTACHMENT表保存任务附件数据，与其对应的实体类AttachmentEntity。

1）实体对象AttachmentEntity

```
protected String id;//对应ID_字段
protected int revision;//附件数据的版本
protected String name;//附件名称
protected String description;//附件描述
protected String type;//附件类型，有调用者定义，保存在TYPE_字段中
protected String taskId;//该附件对应的任务ID
protected String processInstanceId;//流程实例ID
protected String url;//附件的URL
protected String contentId;//附件内容的ID，附件内容会保存到ACT_GE_BYTEARRAY表中
```

2）创建任务附件

主要有两种方式创建附件，一种是通过附件的URL，一种是通过附件内容的输入流inputstream。

```
// 设置任务附件
taskService.createAttachment("web url", task.getId(), pi.getId(), "163.com", 
    "163 web page", "http://www.163.com");

// 创建图片输入流
InputStream is = new FileInputStream(new File("resource/artifact/result.png"));
// 设置输入流为任务附件
taskService.createAttachment("web url", task.getId(), pi.getId(), "163.com", 
    "163 web page", is);
```

注：通过URL设置附件并不会解析URL并声称byte数组，值是单纯的将URL保存到URL_字段中。

3）附件的查询

TaskService提供了4个查询附件相关的方法，包括根据任务ID查询附件集合，根据流程实例ID查询附件集合，根据附件ID查询和根据附件ID查询附件内容。

```
// 设置任务附件
Attachment att1 = taskService.createAttachment("web url", task.getId(), pi.getId(), "Attachement1", "163 web page", "http://www.163.com");
// 创建图片输入流
InputStream is = GetAttachment.class.getResourceAsStream("/artifact/result.png");//new FileInputStream(new File("/artifact/result.png"));
// 设置输入流为任务附件
Attachment att2 = taskService.createAttachment("web url", task.getId(), pi.getId(), "Attachement2", "Image InputStream", is);

// 根据流程实例ID查询附件
List<Attachment> attas1 = taskService.getProcessInstanceAttachments(pi.getId());
System.out.println("流程附件数量：" + attas1.size());//2

// 根据任务ID查询附件
List<Attachment> attas2 = taskService.getTaskAttachments(task.getId());
System.out.println("任务附件数量：" + attas2.size());//2

// 根据附件ID查询附件
Attachment attResult = taskService.getAttachment(att1.getId());
System.out.println("附件1名称：" + attResult.getName());
// 根据附件ID查询附件内容，输入流为null，存储附件的方式是url  
InputStream stream1 = taskService.getAttachmentContent(att1.getId());
System.out.println("附件1的输入流：" + stream1);

InputStream stream2 = taskService.getAttachmentContent(att2.getId());
System.out.println("附件2的输入流：" + stream2);
```

注：存储url方式的任务附件，在使用taskService.getAttachmentContent获取附件的输入流时，返回null。

4）删除附件

```
taskService.deleteAttachment(attchment.getId());
```

注：删除附件方法并不会删除ACT_GE_BYTEARRAY表中的附件内容。