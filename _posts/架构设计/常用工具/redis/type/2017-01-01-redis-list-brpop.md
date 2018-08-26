---
title: redis集合阻塞弹出操作命令
tags: [redis]
---

LPOP和RPOP命令有对应的阻塞版本：BLPOP和BRPOP。

1）可设置阻塞时间

与非阻塞版本类似，阻塞版本的命令也从列表的左端或右端弹出元素；但是，当列表为空时，阻塞版本会将客户端阻塞。我们必须在这些阻塞版本的命令中指定一个以秒为单位的超时时间，表示最长等待几秒。

注：当超时时间为零时，表示永久等待。这个特性在任务调度场景中非常有用；

2）实现任务调度（类似生产者和消费者）

在这种场景下，多个任务执行程序（Redis客户端）会等待任务调度程序分配新的任务。任务执行程序只需要对Redis中的列表使用BLPOP或BRPOP，之后每当有新任务时，调度程序把任务插入到列表中，而任务执行程序之一便会获取到该任务。

3）实践

打开两个终端，它们代表两个Redis客户端，worker-1和worker-2，然后使用redis-cli连接到同一个Redis服务器。在两个任务执行器对应的终端上对同一个列表job_queue执行BRPOP命令，让这两个任务执行器在同一个队列上等待新的任务：

```
# 等待时间设置为0，表示无限等待
worker-1 > brpop job_queue 0

worker-2 > brpop job_queue 0
```

再打开一个终端，用来向job_queue中放置任务

```
# job1会被worker-1拿到，因为worker1先执行的brpop操作
dispatcher > lpush job_queue job1
dispatcher > lpush job_queue job2
```