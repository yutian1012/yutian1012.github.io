---
title: spring boot jpa OneToOne关联保存问题
tags: [spring]
---

问题：在双向关联的基础上的外键保存问题

1）问题代码

user对象先创建出来，此时并不存在Account对象，即并没有关联Account对象。Account对象在使用构造函数中关联上了user对象。使用AccountRepository保存account对象时，发现采用的是级联保存，会先将user对象保存到数据库中，然后再保存account对象。而并没有将account的主键更新到user表的account_id字段中。

```
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
```

后台执行代码输出信息

```
Hibernate: insert into User (account_id, email, nickName, passWord, regTime, userName) values (?, ?, ?, ?, ?, ?)
Hibernate: insert into Account (money, name) values (?, ?)
Hibernate: select account0_.id as id0_, account0_.money as money0_, account0_.name as name0_ from Account acc
```

注：没有设置关联外键时，只执行了2条insert语句

2）解决方法

保存方法前先将所有关系设置好，如account对象通过构造函数设置了user的关联，user使用set方法设置了account的关联。保存时执行逻辑为，使用AccountRepository保存account对象时，发现采用的是级联保存，会先将user对象保存到数据库中，然后再保存account对象，同时user设置了关联关系，会执行update语句，将account的主键设置到use表的account_id字段上。

```
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
    
    //保存外键字段
    u.setAccount(account);
    
    accountRepository.save(account);
    
    Account account2=accountRepository.findByName("zhangsan");
    
    assertNotNull(account2);
    
    System.out.println(account2.getUser().getEmail());
}
```

后台执行代码输出信息（执行前先把@Transactional注解去掉，否则不会保存到数据库中）

```
Hibernate: insert into User (account_id, email, nickName, passWord, regTime, userName) values (?, ?, ?, ?, ?, ?)
Hibernate: insert into Account (money, name) values (?, ?)
Hibernate: update User set account_id=?, email=?, nickName=?, passWord=?, regTime=?, userName=? where id=?
Hibernate: select account0_.id as id0_, account0_.money as money0_, account0_.name as name0_ from Account account0_ where account0_.name=? limit ?
Hibernate: select user0_.id as id3_0_, user0_.account_id as account7_3_0_, user0_.email as email3_0_, user0_.nickName 
```

注：设置了外键关联时，通过update进行更新外键值

3）User端保存

如果从User端保存数据（利用userRepository保存user对象），逻辑上就没有这么麻烦了，只有保存User数据时做好关联就可以正常保存关联信息了，而Account不需要设置user的关联也是没关系的。

```
@Test
@Transactional
public void testSave() {
    Date date = new Date();
    DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
    String formattedDate = dateFormat.format(date);
    Account account=new Account(null,"zhangsan",80000,null);
    
    User u=new User(null,"aa1", "aa123456", "aa@126.com", "aa",formattedDate,account);
    account.setUser(u);
    
    userRepository.save(u);
    
    User u2=userRepository.findByUserName("aa1");
    
    assertNotNull(u2);
    
    assertTrue(u.getId().longValue()==u2.getId().longValue());
    
    System.out.println(u2.getAccount().getName());
}
```

后台执行代码输出

```
Hibernate: insert into Account (money, name) values (?, ?)
Hibernate: insert into User (account_id, email, nickName, passWord, regTime, userName) values (?, ?, ?, ?, ?, ?)
Hibernate: select user0_.id as id3_, user0_.account_id as account7_3_, user0_.email as email3_, user0_.nickName as nickName3
```

注：user端保存，只需要执行2条insert语句即可。