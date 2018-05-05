---
title: spring boot配置使用jdbcTemplate
tags: [spring]
---

spring data只适于JPA方式的操作，不能用到jdbcTemplate上。即使有jdbcTemplate访问数据源不能利用spring data生成代码的功能。

参考：https://blog.csdn.net/forezp/article/details/70477821

1）添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

注：引入start-jdbc会自动注入jdbcTemplate相关的对象到容器中

2）application.properties配置

在配置文件中添加数据源相关配置项。

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

3）数据库创建sql语句

```
-- create table `account`
DROP TABLE `account` IF EXISTS;
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `money` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
INSERT INTO `account` VALUES ('1', 'aaa', '1000');
INSERT INTO `account` VALUES ('2', 'bbb', '1000');
INSERT INTO `account` VALUES ('3', 'ccc', '1000');
```

注：数据库使用自增的方式实现。

4）创建实体类

```
package com.example.demo.model;
public class Account {
    private int id ;
    private String name ;
    private double money;
    //get,set方法
}
```

5）dao层接口及实现类

```
package com.example.demo.dao;

import java.util.List;

import com.example.demo.model.Account;

public interface IAccountDAO {
    int add(Account account);

    int update(Account account);

    int delete(int id);

    Account findAccountById(int id);

    List<Account> findAccountList();
}
```

实现类，添加方法返回数据库生成的主键

```
package com.example.demo.dao;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;

import com.example.demo.model.Account;
import com.mysql.jdbc.Statement;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.List;

/**
 * Created by fangzhipeng on 2017/4/20.
 */
@Repository
public class AccountDaoImpl implements IAccountDAO {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public int add(final Account account) {
        KeyHolder keyHolder = new GeneratedKeyHolder(); 
        
        jdbcTemplate.update(new PreparedStatementCreator() {
            
            @Override
            public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
                PreparedStatement ps = con.prepareStatement("insert into account(name, money) values(?, ?)", Statement.RETURN_GENERATED_KEYS);
                ps.setString(1, account.getName());
                ps.setDouble(2, account.getMoney());
                return ps;
            }
        }, keyHolder);
        
         return keyHolder.getKey().intValue();
        
        /*return jdbcTemplate.update("insert into account(name, money) values(?, ?)",
                  account.getName(),account.getMoney());*/
    }
    @Override
    public int update(Account account) {
        return jdbcTemplate.update("UPDATE  account SET NAME=? ,money=? WHERE id=?",
                account.getName(),account.getMoney(),account.getId());
    }
    @Override
    public int delete(int id) {
        return jdbcTemplate.update("DELETE from account where id=?",id);
    }
    @Override
    public Account findAccountById(int id) {
        List<Account> list = jdbcTemplate.query("select * from account where id = ?", new Object[]{id}, new BeanPropertyRowMapper(Account.class));
        if(list!=null && list.size()>0){
            Account account = list.get(0);
            return account;
        }else{
            return null;
        }
    }
    @Override
    public List<Account> findAccountList() {
        List<Account> list = jdbcTemplate.query("select * from account", new Object[]{}, new BeanPropertyRowMapper(Account.class));
        if(list!=null && list.size()>0){
            return list;
        }else{
            return null;
        }
    }
   
}
```

6）创建测试类

测试类的执行方法要有一定的顺序，使用注解@FixMethodOrder实现。

```
package com.example.demo.dao;

import static org.junit.Assert.*;

import java.util.List;

import javax.annotation.Resource;

import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.example.demo.model.Account;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
@FixMethodOrder(MethodSorters.NAME_ASCENDING)//指定测试方法按名称递增的顺序执行
public class AccountDaoImplTest {

    @Resource
    private AccountDaoImpl accountDaoImpl;
    
    private static int id;//这里必须是静态方法，否则update方法中获取的id值将是0
    
    @Test
    public void test1Add() {
        //id是自增的，因此，也可以不指定值
        Account account=new Account(0,"zhangsan",80000);
        id=accountDaoImpl.add(account);
        assertTrue(id>1);
    }
    
    @Test
    public void test2Update() {
        Account account=accountDaoImpl.findAccountById(id);
        assertNotNull(account);
        
        account.setName("zhangsanchange");
        
        int result=accountDaoImpl.update(account);
        
        assertTrue(result==1);
    }
    
    @Test
    public void test3Delete() {
        int result=accountDaoImpl.delete(id);
        
        assertTrue(result==1);
    }

    @Test
    public void test4FindAccountList() {
        List<Account> list=accountDaoImpl.findAccountList();
        assertNotNull(list);
        assertTrue(list.size()==3);
    }
}
```