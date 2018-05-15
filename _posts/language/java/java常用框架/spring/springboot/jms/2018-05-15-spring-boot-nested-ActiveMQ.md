---
title: 整合内嵌ActiveMQ
tags: [spring]
---

1）引入依赖

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
```

2）配置参数

```
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=50
# 使用发布/订阅模式时，下边配置需要设置成 true
spring.jms.pub-sub-domain=true
```

注：这里不需要指定spring.activemq.broker-url属性

3）在启动类上添加注解@EnableJms

使用嵌入式的ActiveMQ，需要在spring-boot的启动类上添加注解@EnableJms

```
package com.ipph.migratecore;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jms.annotation.EnableJms;

@SpringBootApplication
@EnableJms
public class MigrateCoreApplication {

    public static void main(String[] args) {
        SpringApplication.run(MigrateCoreApplication.class, args);
    }
}
```

4）配置消息队列和订阅

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

5）消息发送者/生产者

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

6）消息接收者/消费者

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

7）测试

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