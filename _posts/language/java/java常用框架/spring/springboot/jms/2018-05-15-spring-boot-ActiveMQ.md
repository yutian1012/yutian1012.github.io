---
title: 整合ActiveMQ
tags: [spring]
---

参考：https://blog.csdn.net/qq727960984/article/details/79667947

参考2：https://blog.csdn.net/liuchuanhong1/article/details/54603546

1）添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<!-- 如果需要配置连接池，添加如下依赖 -->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-pool</artifactId>
</dependency>
<!-- 测试 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.16.0</version>
    <scope>test</scope>
</dependency>
```

2）application.properties中配置消息

```
# activemq 配置
spring.activemq.broker-url=tcp://localhost:61616
spring.activemq.user=admin
spring.activemq.password=admin
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=50
# 使用发布/订阅模式时，下边配置需要设置成 true
spring.jms.pub-sub-domain=true
```

3）添加配置类

```
package com.ipph.migratecore.config;

import javax.jms.Queue;
import javax.jms.Topic;

import org.apache.activemq.command.ActiveMQQueue;
import org.apache.activemq.command.ActiveMQTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JmsConfirguration {
    public static final String QUEUE_NAME = "activemq_queue";
    public static final String TOPIC_NAME = "activemq_topic";
    @Bean
    public Queue queue() {
        return new ActiveMQQueue(QUEUE_NAME);
    }
    @Bean
    public Topic topic() {
        return new ActiveMQTopic(TOPIC_NAME);
    }
}
```

4）生产者类

```
package com.ipph.migratecore.jms;

import javax.jms.Queue;
import javax.jms.Topic;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

@Component
public class JmsSender {
    @Autowired
    private Queue queue;
    @Autowired
    private Topic topic;
    @Autowired
    private JmsMessagingTemplate jmsTemplate;
    
    public void sendByQueue(String message) {
        this.jmsTemplate.convertAndSend(queue, message);
    }
    
    public void sendByTopic(String message) {
        this.jmsTemplate.convertAndSend(topic, message);
    }
}
```

5）消费者类

```
package com.ipph.migratecore.jms;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

import com.ipph.migratecore.config.JmsConfirguration;

@Component
public class JmsReceiver {
    //指定监听的队列
    @JmsListener(destination = JmsConfirguration.QUEUE_NAME)
    public void receiveByQueue(String message) {
        System.out.println("接收队列消息:" + message);
    }
    //指定订阅主题
    @JmsListener(destination = JmsConfirguration.TOPIC_NAME)
    public void receiveByTopic(String message) {
        System.out.println("接收主题消息:" + message);
    }
}
```

6）启动activemq服务

下载activeMQ后，双击解压目录bin下的activemq.bat文件，启动服务。

浏览器访问地址：http://localhost:8161/admin/

7）测试类

```
package com.ipph.migratecore.jms;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class JmsTest {
    @Autowired
    private JmsSender sender;
    
    @Test
    public void testSendByQueue() {
        for(int i=0;i<6;i++) {
            this.sender.sendByQueue("hello activemq queue "+i);
        }
    }
    
    @Test
    public void testSendByTopic() {
        for(int i=0;i<6;i++) {
            this.sender.sendByTopic("hello activemq topic "+i);
        }
    }
}
```