---
title: spring boot测试
tags: [spring]
---

spring测试代码的使用

1）针对普通应用

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class UserRepositoryTest {  ...  }
```

2）针对web应用

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class HelloWorldControlerTests { ... }
```

3）使用assert断言

正确引用junit方法中assertEquals和assertTrue方法。如在测试代码中直接使用断言的方法需要静态导入Assert类的相应方法。

```
import static org.junit.Assert.*;//必须是static
```

4）判断控制台输出

引入jar包

```
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.16.0</version>
    <scope>test</scope>
</dependency>
```

编译测试

```
import static org.junit.Assert.*;

import org.junit.Rule;
import org.junit.Test;
import org.junit.contrib.java.lang.system.SystemOutRule;

public class TestHelloWorld {
    @Rule
    public final SystemOutRule log = new SystemOutRule().enableLog();
    
    @Test
    public void test() {
        log.clearLog();
        System.out.println("HelloWorld!");
        assertTrue(log.getLog().contains("HelloWorld!"));
    }
}
```

5）指定测试运行方法的顺序

junit提供了@FixMethodOrder注解来定义方法的执行顺序。以数据库执行插入，查找，更新为例，方法的执行必须要有一定的顺序，否则不会断言成功。

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
@FixMethodOrder(MethodSorters.NAME_ASCENDING)//指定测试方法按名称递增的顺序执行
public class AccountDaoImplTest {

    @Resource
    private AccountDaoImpl accountDaoImpl;
    
    //这里必须是静态方法，否则update方法中获取的id值将是0
    private static int id;
    
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