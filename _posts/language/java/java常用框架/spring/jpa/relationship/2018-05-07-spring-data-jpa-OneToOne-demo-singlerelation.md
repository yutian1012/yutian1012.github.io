---
title: spring boot jpa OneToOne单向关联
tags: [spring]
---

### 单向关联

1）问题：关联字段存储位置（外键存储在使用@OneToOne注解的一端）

单向关联有一个问题，把关联字段存放到那个实体中呢？在表设计中存储到哪个实体中都很合适，但在JPA方式中却不是这样的。在类中属性添加@OneToOne为主控方，外键存放到该实体表中。

2）单向关联的实现

单向关联：即从User实体中获取Account，而Account并不需要获取User，或者方向相反。

如果我们想通过User类获取Account实体，那么会在User类中添加Account实体属性，那么通过get方法就可以实现相关单向数据的查询了。而在Account属性上设置@OneToOne，标识关联实体对象。设置@JointColumn指定外键信息。

3）修改User实体类

即添加关联对象，在User实体类中添加Account属性，而Account不用修改

```
@OneToOne(optional=true)
@JoinColumn(name="account_id")
private Account account;
```

4）修改测试类

即在保存User对象时同时保持Account对象

```
public void testSave() {
    Date date = new Date();
    DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
    String formattedDate = dateFormat.format(date);
    Account account=new Account(null,"zhangsan",80000);
    User u=new User(null,"aa1", "aa123456", "aa@126.com", "aa",formattedDate,account);
    
    userRepository.save(u);
    
    User u2=userRepository.findByUserName("aa1");
    
    assertNotNull(u2);
    
    assertTrue(u.getId().longValue()==u2.getId().longValue());

    assertNotNull(u2.getAccount());
}
```

5）异常信息

```
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.TransientPropertyValueException: object references an unsaved transient instance - save the transient instance before flushing...
```

大概的意思是要保存user对象，需要先保存account对象。

6）解决方法，这里提供2种解决方法

方式一：先保存Account对象，使其成为持久化对象，然后再保存User对象。

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