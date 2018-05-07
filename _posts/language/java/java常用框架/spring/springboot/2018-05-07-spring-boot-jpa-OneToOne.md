---
title: spring boot jpa OneToOne
tags: [spring]
---

参考：https://blog.csdn.net/lyg_2012/article/details/70195062

@OneToOne的一个实用场景，一个表包含了很多的Text大文本字段，而实际在关联查询时并不需要这些数据，直接查询很消耗性能，一般需要间将这部分数据独立成一个关联表，真正使用时再去查询。

以用户和账号为例，这里假定用户和账号是一一对应的，即一个用户只有一个账号，同理一个账号也只属于一个用户。

1）用户字段信息

```
package com.example.demo.model;

import java.io.Serializable;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToOne;

import lombok.Data;

@Entity
@Data
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
    //级联更新
    @OneToOne(cascade=CascadeType.MERGE)
    @JoinColumn(name="account_id")
    private Account account;

    public User(String userName, String passWord, String email, String nickName, String regTime) {
        super();
        this.userName = userName;
        this.passWord = passWord;
        this.email = email;
        this.nickName = nickName;
        this.regTime = regTime;
    }
}
```

2）用户账号信息

```
package com.example.demo.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Data;

@Entity
@Data
@AllArgsConstructor
public class Account {
    @Id
    @GeneratedValue
    private int id ;
    @Column
    private String name ;
    @Column
    private double money;
}
```

3）查看生成的表结构

```
mysql> desc user;
+------------+--------------+------+-----+---------+-------+
| Field      | Type         | Null | Key | Default | Extra |
+------------+--------------+------+-----+---------+-------+
| id         | bigint(20)   | NO   | PRI | NULL    |       |
| email      | varchar(255) | NO   | UNI | NULL    |       |
| nick_name  | varchar(255) | YES  | UNI | NULL    |       |
| pass_word  | varchar(255) | NO   |     | NULL    |       |
| reg_time   | varchar(255) | NO   |     | NULL    |       |
| user_name  | varchar(255) | NO   | UNI | NULL    |       |
| account_id | int(11)      | YES  | MUL | NULL    |       |
+------------+--------------+------+-----+---------+-------+

mysql> desc account;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| id    | int(11)      | NO   | PRI | NULL    |       |
| money | double       | YES  |     | NULL    |       |
| name  | varchar(255) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
```

注：在user表中存在account_id字段关联account。

4）dao层类

a.用户实体存储dao

```
package com.example.demo.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.demo.model.User;

public interface UserRepository extends JpaRepository<User, Long> { 
    User findByUserName(String userName);    
    User findByUserNameOrEmail(String username, String email);
}
```

b.账号实体存储dao

```
package com.example.demo.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.demo.model.Account;

public interface AccountRepository extends JpaRepository<Account, Long>{

}
```

5）测试类

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

import com.example.demo.model.Account;
import com.example.demo.model.User;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class UserRepositoryTest {    
    @Autowired
    private UserRepository userRepository;   
    
    @Test
    public void testAdd() {
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
        String formattedDate = dateFormat.format(date);
        User u=new User("aa1", "aa@126.com", "aa", "aa123456",formattedDate);
        
        Account account=new Account("zhangsan",80000);
        
        u.setAccount(account);
        
        userRepository.save(u);        
    }

}
```

异常信息

```
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.TransientPropertyValueException: object references an unsaved transient instance - save the transient instance before flushing...
```

大概的意思是要保存user对象，需要先保存account对象。

解决方法，这里提供2种解决方法

方式一：先保存Account对象，然后再保存User对象。

```
accountRepository.save(account);
u.setAccount(account);
userRepository.save(u);
```

注：这里只提供了主要的代码。

方式二：修改级联操作方式，这里修改为级联保存

```
@OneToOne(cascade=CascadeType.PERSIST)
@JoinColumn(name="account_id")
private Account account;
```

6）思考

这里主要的记录是User表，而account是与user表关联的，可以有user而可以没有