---
title: spring boot jpa OneToOne无关联
tags: [spring]
---

@OneToOne的一个实用场景，一个表包含了很多的Text大文本字段，而实际在关联查询时并不需要这些数据，直接查询很消耗性能，一般需要间将这部分数据独立成一个关联表，真正使用时再去查询。

以用户和账号为例，这里假定用户和账号是一一对应的，即一个用户只有一个账号，同理一个账号也只属于一个用户。

这里不作关联设置，一对一的单向关联和双向关联在下一个文件中介绍，这里做一个基础类。

1）实体类

a.用户实体类字段信息

```
package spittr.domain;

import java.io.Serializable;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToOne;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
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
}
```

注：添加

b.用户账号实体类字段信息

```
package spittr.domain;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Account implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue
    private Long id ;
    @Column
    private String name ;
    @Column
    private double money;
}
```

2）dao层类

a.用户实体存储dao

```
package spittr.db;

import org.springframework.data.jpa.repository.JpaRepository;
import spittr.domain.User;

public interface UserRepository extends JpaRepository<User, Long> { 
    User findByUserName(String userName);    
    User findByUserNameOrEmail(String username, String email);
}
```

b.账号实体存储dao

```
package spittr.db;

import org.springframework.data.jpa.repository.JpaRepository;
import spittr.domain.Account;

public interface AccountRepository extends JpaRepository<Account, Long>{
    Account findByName(String name);
}
```

3）测试类

a.User实体类的测试

```
package spittr.db;

import static org.junit.Assert.*;

import java.text.DateFormat;
import java.util.Date;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import spittr.domain.User;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JpaConfig.class)
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;
    @Test
    @Transactional
    public void testSave() {
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
        String formattedDate = dateFormat.format(date);
        User u=new User(null,"aa1", "aa123456", "aa@126.com", "aa",formattedDate);
        userRepository.save(u);
        
        User u2=userRepository.findByUserName("aa1");
        assertNotNull(u2);
        assertTrue(u.getId().longValue()==u2.getId().longValue());
    }
}
```

b.Account实体类的测试

```
package spittr.db;

import static org.junit.Assert.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import spittr.domain.Account;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JpaConfig.class)
public class AccountRepositoryTest {

    @Autowired
    private AccountRepository accountRepository;
    
    @Test
    @Transactional
    public void test() {
        Account account=new Account(null,"zhangsan",80000);
        accountRepository.save(account);
        Account account2=accountRepository.findByName("zhangsan");
        
        assertNotNull(account2);
        assertTrue(account.getId().longValue()==account2.getId().longValue());
    }
}
```