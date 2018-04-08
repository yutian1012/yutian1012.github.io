---
title: spring数据访问与集成
tags: [spring]
---

数据集成模块包含了JDBC，Transaction，ORM，OXM，Messaging，JMS等jar包

1）简化样板代码编写，封装异常错误。

使用JDBC编写代码通常会导致大量的样板式代码，例如获得数据库连接、创建语句、处理结果集到最后关闭数据库连接。Spring的JDBC和DAO（DataAccessObject）模块抽象了这些样板式代码，使我们的数据库代码变得简单明了，还可以避免因为关闭数据库资源失败而引发的问题。该模块在多种数据库服务的错误信息之上构建了一个语义丰富的异常层，以后我们再也不需要解释那些隐晦专有的SQL错误信息了！使用SpringAOP模块为Spring应用中的对象提供事务管理服务。

2）提供了丰富的ORM和事务支持

对于那些更喜欢ORM（Object-RelationalMapping）工具而不愿意直接使用JDBC的开发者，Spring提供了ORM模块。Spring的ORM模块建立在对DAO的支持之上，并为多个ORM框架提供了一种构建DAO的简便方式。Spring没有尝试去创建自己的ORM解决方案，而是对许多流行的ORM框架进行了集成，包括Hibernate、JavaPersisternceAPI、JavaDataObject和iBATISSQLMaps。Spring的事务管理支持所有的ORM框架以及JDBC。

3）JMS抽象封装处理

本模块同样包含了在JMS（Java Message Service）之上构建的Spring抽象层，它会使用消息以异步的方式与其他应用集成。

4）java对象与xml对象的转换

### jar包信息

1）spring-jdbc.jar

对JDBC数据访问进行封装。

2）spring-jms.jar

为简化JMS API的使用而作的简单封装

3）spring-orm.jar

整合第三方的ORM框架，如hibernate,ibatis,jdo，以及spring的JPA实现。

4）spring-oxm.jar

Spring 对Object/XMl的映射支持,可以让Java与XML之间来回切换。

5）spring-transaction.jar

为JDBC、Hibernate、JDO、JPA等提供的一致的声明式和编程式事务管理

6）spring-messaging.jar

该jar是spring4.0为了集成jms发布的一个新模块，只有spring 4.0以后才支持。

新的消息（messaging）模块，很多的类型来源于Spring-Integration项目。这个消息模块支持Spring的SockJS/STOMP功能，同时提供了基于模板的方式发布消息。

