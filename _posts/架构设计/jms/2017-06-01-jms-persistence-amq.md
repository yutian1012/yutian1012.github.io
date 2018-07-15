---
title: activeMQ的AMQ持久化方式（不推荐使用）
tags: [java_middleware]
---

AMQ适用于ActiveMQ5.3之前的版本。

### AMQ（Advanced Message Queuing）

参考：http://activemq.apache.org/amq-message-store.html

性能高于JDBC，写入消息时，会将消息写入日志文件，由于是顺序追加写，性能很高。为了提升性能，创建消息主键索引，并且提供缓存机制，进一步提升性能。每个日志文件的大小都是有限制的（默认32m，可自行配置）。当超过这个大小，系统会重新建立一个文件。当所有的消息都消费完成，系统会删除这个文件或者归档（取决于配置）。主要的缺点是AMQ Message会为每一个Destination创建一个索引，如果使用了大量的Queue，索引文件的大小会占用很多磁盘空间。而且由于索引巨大，一旦Broker崩溃，重建索引的速度会非常慢。

虽然AMQ性能略高于Kaha DB，但是由于其重建索引时间过长，而且索引文件占用磁盘空间过大，所以已经不推荐使用。这里就不在详细介绍AMQ持久化的数据结构。在新版本的ActiveMQ中，AMQ已经被删除。

注：日志文件中记录消息。

![](/images/middleware/jms/activemq-persistence-amq.png)

配置方式

```
<persistenceAdapter>  
     <amqPersistenceAdapter directory="${activemq.data}/activemq-data" maxFileLength="32mb"/>
</persistenceAdapter> 
```

错误信息

```
发现了以元素 'amqPersistenceAdapter' 开头的无效内容。
```

原因：使用版本为apache-activemq-5.9.1。

### 目录结构

下载使用apache-activemq-5.1.0，查看data目录结构（启动后）

![](/images/middleware/jms/activemq-persistence-amq-data.png)

```
D:\WORK\INSTALLER\APACHE-ACTIVEMQ-5.1.0\DATA
│  activemq.log
│  lock
│
├─journal
│      data-1
│      data-control
│
├─kr-store
│  ├─data
│  │      data-container-roots-1
│  │      hash-index-queue-data_queue#3a#2f#2fexample.A
│  │      index-container-roots
│  │      index-queue-data
│  │      lock
│  │
│  └─state
│          data-kaha-1
│          data-store-state-1
│          hash-index-store-state_state
│          index-kaha
│          index-store-state
│          index-transactions-state
│          lock
│
└─localhost
    └─tmp_storage
```

1）archive

放置移除或归档的消息日志文件

2）journal

放置消息日志信息（即消息日志文件的存储目录）

3）kr-store

持久化访问为KahaDB时，存放（kr Kaha reference）消息内容

4）tmp-storage

类似内存的swap（交换区），减轻内存的消耗。