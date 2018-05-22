---
title: spring data Jpa设置自定查询方法--混合自定义查询
tags: [spring]
---

有些时候，我们需要Repository所提供的功能是无法用SpringData的方法命名约定来描述的，甚至无法用@Query注解设置查询来实现。尽管Spring Data JPA非常棒，但是它依然有其局限性，可能需要我们按照传统的方式来编写Repository方法：也就是直接使用EntityManager。

1）spring data提供的解决方案

只需在类路径下提供一个与Repository接口类相同，并以Impl为后者的类即可实现混合定义方法。

当Spring Data JPA为Repository接口生成实现的时候，它还会查找名字与接口相同，并且添加了Impl后缀的一个类。如果这个类存在的话，SpringDataJPA将会把它的方法与SpringDataJPA所生成的方法合并在一起。

2）实例（在jpa-demo的基础上实现）

条件1：为混合类提供一个单独的接口，混合类（即Impl为后缀的类）实现该接口。

条件2：Repository接口相关的类也要实现这个新接口，目的是提供与新接口相同的方法。

条件3：混合类的名称与Repository接口名称要匹配，后缀为Impl（可配置）

注：从实现上可以看出，内部是使用代理来解决的，通过代理接口来将方法的调用定位到实现类方法的。

a.新接口

```
package spittr.db;

import spittr.domain.Spitter;

public interface SpitterExtentionRepository {
    
    /**
     * 提供一个spring data jpa扩展方法，Gmail并发Spitter的属性
     * @param gMail
     * @return
     */
    public Spitter getSpitterByGmail(String gMail);
}
```

b.Repository接口

```
package spittr.db;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;

import spittr.domain.Spitter;

/**
 * Repository interface with operations for {@link Spitter} persistence.
 * @author habuma
 */
public interface SpitterRepository extends JpaRepository<Spitter, Long> ,SpitterExtentionRepository{
      
    Spitter findByUsername(String username);
    
    List<Spitter> findByUsernameOrFullNameLike(String username, String fullName);
}
```

c.实现类

```
package spittr.db;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;

import spittr.domain.Spitter;

public class SpitterRepositoryImpl implements SpitterExtentionRepository{
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public Spitter getSpitterByGmail(String gMail) {
        String sql="select spitter from Spitter as spitter where spitter.email=?1";
        Query query=em.createQuery(sql);
        query.setParameter(1, gMail);
        return (Spitter) query.getSingleResult();
    }

}
```

3）测试代码

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
public class SpitterExtentionRepositoryTest {

    @Autowired
    SpitterRepository spitterRepository;
    
    @Test
    @Transactional
    public void testSave() {
        Spitter spitter = new Spitter(null, "newbee", "letmein", "New Bee", "newbee@Gmail.com", true);
        spitterRepository.save(spitter);
        Spitter spitter2=spitterRepository.getSpitterByGmail("newbee@Gmail.com");
        assertNotNull(spitter);
        assertTrue(spitter.getId().longValue()==spitter2.getId());
    }
}
```