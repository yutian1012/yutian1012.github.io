---
title: spring boot jpa OneToMany无关联
tags: [spring]
---

在下单时，用户会选择包裹配送地址（多个地址选择其一），这里就涉及到了用户与地址的关系。用户与地址的关系属于一对多的关系，即一个用户存在多个地址，而每个地址只会属于一个用户。

1）实体类

a.用户沿用一对一模块的User类

```
package spittr.domain;

import java.io.Serializable;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
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
    @OneToOne(optional=true,cascade=CascadeType.PERSIST,fetch=FetchType.LAZY)
    @JoinColumn(name="account_id")
    private Account account;
}
```

b.地址类

```
package spittr.domain;

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
public class Address {
    @Id
    @GeneratedValue
    private Long id;
    private String address;
    private String zipcode;
    private String city;
}
```

2）dao层类

a.User类的dao层操作类

```
package spittr.db;

import org.springframework.data.jpa.repository.JpaRepository;

import spittr.domain.User;

public interface UserRepository extends JpaRepository<User, Long> { 
    User findByUserName(String userName);    
    User findByUserNameOrEmail(String username, String email);
}
```

b.Address类的dao层操作类

```
package spittr.db;

import org.springframework.data.jpa.repository.JpaRepository;

import spittr.domain.Address;

public interface AddressRepository extends JpaRepository<Address, Long>{
    
}
```

3）数据保存测试

```
package spittr.db;

import static org.junit.Assert.*;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import spittr.domain.Address;
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JpaConfig.class)
public class AddressRepositoryTest {

    @Resource
    private AddressRepository addressRepository;
    
    @Test
    @Transactional
    public void testSave() {
        Address address=new Address(null, "北京海淀区", "10000", "北京");
        
        addressRepository.save(address);
        
        Long id=address.getId();
        
        assertNotNull(id);
    }
}
```