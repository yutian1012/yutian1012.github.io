---
title: activeMQ简介
tags: [java_middleware]
---

### 下载并运行软件

1）下载

下载地址：http://activemq.apache.org/

下载完成后解压到D:\work\installer\apache-activemq-5.9.1目录下。并运行D:\sjq\apache-activemq-5.9.1\bin\activemq.bat文件

![](/images/middleware/jms/activemq-start.png)

注：在启动前要设置好java的运行环境，JAVA_HOME环境变量

注2：启动时，启动了内置的jetty服务器

2）测试

在浏览器中访问：http://localhost:8161/admin/

输入用户名和密码（用户名和密码都是admin）

注：在D:\sjq\apache-activemq-5.9.1\conf\users.properties文件中可以查看

![](/images/middleware/jms/activemq-admin.png)

### 管理界面创建queue

![](/images/middleware/jms/activemq-admin-createqueue.png)

注：上面创建了一个名为firstQueue队列。

### activemq的API

![](/images/middleware/jms/jms-activemq-api.png)

执行步骤：

1）创建jms连接工厂

2）创建连接

3）创建session，发送或接收消息的线程

4）创建消息的目的地，即消息发送到什么地方

5）创建消息发送者或消费者

6）使用消息生产者发送消息，或者使用消费者接收消息。

### 创建项目

1）引入相关jar包

```
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-client</artifactId>
    <version>5.9.1</version>
</dependency>
```

2）创建消息发送者

![](/images/middleware/jms/activemq-connection.png)

注：启动信息中能看到连接队列的地址tcp开头。

```
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.DeliveryMode;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
public class Producer {
    public static void main(String[] args) {
        //创建jms连接工厂
        ConnectionFactory connectionFactory;
        //创建连接
        Connection connection=null;
        //创建session，发送或接收消息的线程
        Session session=null;
        //消息的目的地，即消息发送给谁
        Destination destination;
        //消息发送者,即生产者
        MessageProducer producer;
        
        //初始化各个消息机制的组件
        connectionFactory=new ActiveMQConnectionFactory(ActiveMQConnection.DEFAULT_USER,ActiveMQConnection.DEFAULT_PASSWORD,"tcp://localhost:61616");
        
        try{
            //通过工厂创建连接对象
            connection=connectionFactory.createConnection();
            //启动连接
            connection.start();
            //获取session指定是否使用消息的事务机制，采用何种方式的确认机制
            session=connection.createSession(true,Session.AUTO_ACKNOWLEDGE);
            //指定消息发送的目标队列
            destination=session.createQueue("firstQueue");//为目标生成一个标识
            //消息生成者
            producer=session.createProducer(destination);//指定生产的消息放入的目标位置
            //设置消息是否为持久化消息,这里设置为非持久化消息，即不向磁盘存储
            producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            //发送消息
            sendMessage(session,producer);
            //提交事务
            session.commit();
        }
        catch(Exception e){
            e.printStackTrace();
        }
        finally{
            if(null!=connection){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    /**
     * 发送消息的代码
     * @param session
     * @param producer
     * @throws JMSException 
     */
    public static void sendMessage(Session session,MessageProducer producer) throws JMSException{
        int send_number=5;//发送5个消息
        for(int i=0;i<send_number;i++){
            //构建消息体
            TextMessage message=session.createTextMessage("ActiveMQ发送的第"+(i+1)+"个消息");
            System.out.println("ActiveMQ发送的第"+(i+1)+"个消息");
            //发送消息
            producer.send(message);
        }
    }
}
```

注：运行生产者类，会向队列发送5个消息。

![](/images/middleware/jms/activemq-admin-queue-message.png)

再次运行Producer类，会发现消息变成了10条（向队列中又加入了5条消息）

注：destination=session.createQueue("firstQueue");如果服务器中存在该队列，则不需要继续创建，直接使用。

3）消息消费者

```
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

public class Consumer {
    public static void main(String[] args) {
        //创建jms连接工厂
        ConnectionFactory connectionFactory;
        //创建连接
        Connection connection=null;
        //创建session，发送或接收消息的线程
        Session session=null;
        //消息的目的地，即消息发送给谁
        Destination destination;
        //消息接收者，即消费者
        MessageConsumer consumer;
        
        //初始化各个消息机制的组件
        connectionFactory=new ActiveMQConnectionFactory(ActiveMQConnection.DEFAULT_USER,ActiveMQConnection.DEFAULT_PASSWORD,"tcp://localhost:61616");
        
        try{
            //通过工厂创建连接对象
            connection=connectionFactory.createConnection();
            //启动连接
            connection.start();
            //获取session指定是否使用消息的事务机制，采用何种方式的确认机制
            session=connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
            //指定消息发送的目标队列
            destination=session.createQueue("firstQueue");//为目标生成一个标识
            //消息消费者-指定生产的消息放入的目标位置
            consumer=session.createConsumer(destination);
            while(true){
                //消费者会一直运行100s，然后停止运行。设置100秒的超时时间
                TextMessage message=(TextMessage) consumer.receive(100*1000);
                if(null!=message){
                    System.out.println("收到的消息："+message.getText());
                }else{
                    break;
                }
            }
        }
        catch(Exception e){
            e.printStackTrace();
        }
        finally{
            if(null!=connection){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```