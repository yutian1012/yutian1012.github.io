---
title: spring boot jpa OneToOne双向关联
tags: [spring]
---

### 双向关联

1）双向关联的实现

双向关联：即能从User实体中获取Account，同时也可也从Account实体中获取User对象，即两个方向上都可以实现关联对象的获取。

User类中存在account对象的属性，同时Account类中也存在User对象的属性。并且两侧都使用@OneToOne注解。并且主控方（外键存储在主控方），另外一方使用的@OneToOne注解要使用mappedBy指定对方相应的属性（放置再次生成外键）。

2）修改Account类

在Account类上添加User属性，并设置mappedBy属性，该属性为User类的account属性名，而不是account_id（表字段名）。

```
@OneToOne(cascade=CascadeType.PERSIST,optional=true,mappedBy="account")
private User user;
```

3）测试类

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

import spittr.domain.Account;
import spittr.domain.User;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JpaConfig.class)
public class AccountRepositoryTest {

    @Autowired
    private AccountRepository accountRepository;
    
    @Test
    @Transactional
    public void test() {
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
        String formattedDate = dateFormat.format(date);
        //user对象设置account为null
        User u=new User(null,"aa1", "aa123456", "aa@126.com", "aa",formattedDate,null);
        
        //关联上User对象
        Account account=new Account(null,"zhangsan",80000,u);
        accountRepository.save(account);
        
        Account account2=accountRepository.findByName("zhangsan");
        
        assertNotNull(account2);
    }
}
```