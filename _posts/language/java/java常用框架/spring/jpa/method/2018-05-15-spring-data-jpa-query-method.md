---
title: spring data Jpa设置自定查询方法
tags: [spring]
---

Spring Data的DSL依旧有其局限性，有时候通过方法名称表达预期的查询很烦琐，甚至无法实现。如果遇到这种情形的话，Spring Data能够让我们通过@Query注解来解决问题。

这种查询可以声明在Repository接口方法中，摆脱像命名查询那样的约束，将查询直接在相应的接口方法中声明，结构更为清晰，这是Spring data的特有实现。 

@Query注解方式能够使用参数索引和参数名的方式实现查询。

1）实例代码

在接口的方法上添加注解@Query，并定义查询sql，如方法findSpitterByUsernameInfo

```
package spittr.db;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import spittr.domain.Spitter;

/**
 * Repository interface with operations for {@link Spitter} persistence.
 * @author habuma
 */
public interface SpitterRepository extends JpaRepository<Spitter, Long>{
      
    Spitter findByUsername(String username);
    
    List<Spitter> findByUsernameOrFullNameLike(String username, String fullName);
    
    //使用索引方式查询
    @Query("select p from Spitter as p where p.username=?1")
    Spitter findSpitterByUsernameInfo(String username);

    //使用参数名匹配查询
    @Query("select p from Spitter as p where p.username=:username")
    Spitter findSpitterByUsernameParamName(@Param("username")String username);
}
```

注：索引参数如下所示，索引值从1开始，查询中”?X”个数需要与方法定义的参数个数相一致，并且顺序也要一致。

2）测试类

```
package spittr.db;

import static org.junit.Assert.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import spittr.domain.Spitter;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JpaConfig.class)
public class SpitterRepositoryQueryTest {

    @Autowired
    SpitterRepository spitterRepository;
    
    @Test
    @Transactional
    public void testSave() {
        Spitter spitter = new Spitter(null, "newbee", "letmein", "New Bee", "newbee@habuma.com", true);
        spitterRepository.save(spitter);

        Spitter spitter2=spitterRepository.findSpitterByUsernameInfo("newbee");
        assertNotNull(spitter2);
        assertTrue(spitter.getId().longValue()==spitter2.getId().longValue());

        Spitter spitter3=spitterRepository.findSpitterByUsernameParamName("newbee");
        assertNotNull(spitter3);
        assertTrue(spitter.getId().longValue()==spitter3.getId().longValue());
    }
}
```