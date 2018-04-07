---
title: spring不同环境下的数据库配置
tags: [spring]
---

1）测试环境下配置嵌入式数据库

参考：https://blog.csdn.net/xwq911/article/details/51536156

EmbeddedDatabaseBuilder创建的DataSource非常适于开发环境。当你在开发环境中运行集成测试或者启动应用进行手动测试的时候，可以使用嵌入式数据库，这样每次启动它的时候，都能让数据库处于一个给定的状态。

使用EmbeddedDatabaseBuilder会搭建一个嵌入式的Hypersonic数据库，它的模式（schema）定义在schema.sql中，测试数据则是通过test-data.sql加载的。

```
//创建一个类型为javax.sql.DataSource的bean
@Bean(destroyMethod="shutdown")
public DataSource dataSource(){
    return new EmbeddedDatabaseBuilder().type(EmbeddedDatabaseType.H2)
        .addScript("classpath:schema.sql")
        .addScript("classpath:test-data.sql").build();
}
```

注：这里的EmbeddedDatabaseBuilder类都是jdbc包中提供的，要使用具体的数据库，还需要提供相应的jar

```
<!-- Spring JDBC -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>${spring.version}</version>
</dependency>

<!-- H2 DB -->
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>${h2.version}</version>
</dependency>
```

2）生产环境中配置数据库

在生产环境的配置中，你可能会希望使用JNDI从容器中获取一个DataSource。通过JNDI获取DataSource能够让容器决定该如何创建这个DataSource，甚至包括切换为容器管理的连接池。

```
@Bean
public DataSource dataSource(){
    JndiObjectFactoryBean jndiObjectFactoryBean=new JndiObjectFactoryBean();
    jndiObjectFactoryBean.setJndiName("jdbc/myDS");
    jndiObjectFactoryBean.setResourceRef(true);
    jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
    return(DataSource)jndiObjectFactoryBean.getObject();
}
```

注：JndiObjectFactoryBean也是由jdbc包提供的。

tomcat目录conf/context.xml文件

```
<Resource name="jdbc/mmcDB" //指定的jndi名称
    auth="Container"//认证方式，一般默认这个
    type="javax.sql.DataSource" //数据源床型，使用标准的javax.sql.DataSource
    driverClassName="com.mysql.jdbc.Driver"    //JDBC驱动器 
    url="jdbc:mysql://localhost:3306/test" //数据库URL地址             
    username="test"     //数据库用户名
    password="test"   //数据库密码
    maxIdle="40"   //最大的空闲连接数
    maxWait="4000" //当池的数据库连接已经被占用的时候，最大等待时间
    maxActive="250" //连接池当中最大的数据库连接
    removeAbandoned="true" 
    removeAbandonedTimeout="180"
    logAbandoned="true" //被丢弃的数据库连接是否做记录，以便跟踪
    factory="org.apache.tomcat.dbcp.dbcp.BasicDataSourceFactory" />
```

3）QA环境

在QA环境中，你可以选择完全不同的DataSource配置，可以配置为Commons DBCP连接池。

```
@Bean(destroyMethod="close")
public DataSource dataSource(){
    BasicDataSource dataSource=new BasicDataSource();
    dataSource.setUrl("jdbc:h2:tcp://dbserver/~/test");
    dataSource.setDriverClassName("org.h2.Driver");
    dataSource.setUsername("sa");
    dataSource.setPassword("password");
    dataSource.setInitialSize(20);
    dataSource.setMaxActive(30);
    returndataSource;
}
```

注：这里除了要引入h2数据库的依赖jar，还需要引入commons DBC连接池。

```
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
```