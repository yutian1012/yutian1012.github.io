---
title: spring boot配置使用JPA
tags: [spring]
---

spring Data JPA能够自动生成实现方法，通过特殊方法约定来实现。

jap的查询包括：方法命名查询，@NamedQuery查询和@Query查询三种方式查询。

1）添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

2）application.properties配置

在配置文件中添加数据源以及hibernate相关配置项。

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql= true
```

hibernate.hbm2ddl.auto参数的作用取值create，create-drop，update以及validate。其中update是最常用的，每次都会根据实体类来更新数据表结构。

注：show-sql主要是打印出自动生产的SQL，方便调试的时候查看。

3）创建实体类

这里的注解都是唯一javax.persistence包下，即JPA规范的注解。并且实体类要实现Seriablizable接口。

实体类必须提供无参构造函数，因为jpa在创建对象时会使用无参构造创建对象，然后在对相应的属性赋值。

```
package com.example.demo.model;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class User implements Serializable { 
    private static final long serialVersionUID = 1L;    
    
    @Id
    @GeneratedValue
    private Long id;    
    @Column(nullable = false, unique = true)    
    private String userName;    
    @Column(nullable = false)    
    private String passWord;    
    @Column(nullable = false, unique = true)    
    private String email;    
    @Column(nullable = true, unique = true)    
    private String nickName;    
    @Column(nullable = false)    
    private String regTime; 

    //提供一个无参构造函数，hibernate封装对象时会调用无参构造
    public User(){};

    public User(String userName, String email,String nickName,String passWord,  String regTime) {
        this.userName = userName;
        this.email = email;
        this.nickName = nickName;
        this.passWord = passWord;
        this.regTime = regTime;
    }

    //set/get method...
}
```

注：使用lombok可以不用设置set和get方法，并且能够自动生成相应的构造函数。

4）创建dao层类

利用spring data jpa功能自动生成实现类。

```
package com.example.demo.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.demo.model.User;

public interface UserRepository extends JpaRepository<User, Long> { 
    User findByUserName(String userName);    
    User findByUserNameOrEmail(String username, String email);
}
```

5）测试dao层方法

```
package com.example.demo.dao;

import java.text.DateFormat;
import java.util.Date;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.example.demo.model.User;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class UserRepositoryTest {    
    @Autowired
    private UserRepository userRepository;   
    @Test
    public void test() throws Exception {
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
        String formattedDate = dateFormat.format(date);

        userRepository.save(new User("aa1", "aa@126.com", "aa", "aa123456",formattedDate));
        userRepository.save(new User("bb2", "bb@126.com", "bb", "bb123456",formattedDate));
        userRepository.save(new User("cc3", "cc@126.com", "cc", "cc123456",formattedDate));

        Assert.assertEquals(3, userRepository.findAll().size());
        Assert.assertEquals("bb", userRepository.findByUserNameOrEmail("bb2", "ccw@126.com").getNickName());
        userRepository.delete(userRepository.findByUserName("aa1"));
    }
}
```

6）运行

```
mvn test
```

类路径下必须要存在@SpringBootApplication主键的配置类，该类是spring boot的启动类

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

错误信息

```
Incorrect string value: '\xE5\xB9\xB45\xE6\x9C...' for column 'reg_time'
```

该情况一般是由数据库设计时的编码错误导致的。

```
mysql> show variables like 'character%';
+--------------------------+-------------------+
| Variable_name            | Value|
+--------------------------+---------------------+
| character_set_client     | utf8|
| character_set_connection | utf8|
| character_set_database   | latin1|
| character_set_filesystem | binary|
| character_set_results    | utf8|
| character_set_server     | utf8|
| character_set_system     | utf8    
```

注：上面的charater_set_database的编码是latin1。

解决方法，单独修改test库的编码

```
alter database `test` character set utf8;
```

修改完成后，删除hibernate创建的user表，重新生成。