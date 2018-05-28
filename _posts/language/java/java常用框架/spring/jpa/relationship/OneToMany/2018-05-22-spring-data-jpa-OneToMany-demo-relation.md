---
title: spring boot jpa OneToMany关联实现
tags: [spring]
---

完成从用户关联地址集合的操作，即通过用户查询用户的地址集合。

不论@JoinColumn注解放置到哪个实体类上，最终都会在Address表中生成外键，即在多的一端维护关联。

1）修改模型类

a.User类

```
@OneToMany(cascade=CascadeType.REMOVE)
@JoinColumn(name="user_id")
private List<Address> addressList;
```

注：当用户删除了，那么Address也没有意义了，这里设置了级联删除

b.Address类

```
@ManyToOne
private User user;
```

2）修改测试类

添加测试类，实际业务场景是这样的。一般用户先注册，注册时并不需要填写地址信息，当用户需要购买物品时，提交订单，完善送货地址时才需要设置地址。

```
package spittr.db;

import static org.junit.Assert.*;

import java.text.DateFormat;
import java.util.Date;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import spittr.domain.Address;
import spittr.domain.User;
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JpaConfig.class)
public class AddressRepositoryTest {

    @Resource
    private AddressRepository addressRepository;
    
    @Resource
    private UserRepository userRepository;
    
    @Test
    @Transactional
    public void testSave() {
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
        String formattedDate = dateFormat.format(date);
        
        User user=new User();
        user.setUserName("test");
        user.setRegTime(formattedDate);
        user.setNickName("t");
        user.setPassWord("xxxxx");
        user.setEmail("test@qq.com");
        
        userRepository.save(user);//先注册用户完成
        
        //用户填写地址1
        Address address=new Address(null, "北京海淀区", "10000", "北京",user);
        
        addressRepository.save(address);
        
        Long addressId=address.getId();
        
        //用户填写地址2
        Address address2=new Address(null, "北京海淀区2", "10002", "北京2",user);
        
        addressRepository.save(address2);
        
        //从数据库中加载用户
        User dbUser=userRepository.findByUserName("test");
        
        assertTrue(2==dbUser.getAddressList().size());
        
        //获取地址，并判断关联用户
        Address add=addressRepository.findOne(addressId);
        
        assertEquals("test", add.getUser().getUserName());
    }
}
```

3）错误

注：在测试类上的添加事务，并不会真正将数据提交到数据库中，因此运行完单元测试后，数据库仍然是空值。

4）解决方法

创建service层，保存数据到数据库中，然后再次查询

a.AddressService类

```
package spitter.service;

import javax.annotation.Resource;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import spittr.db.AddressRepository;
import spittr.domain.Address;

@Service
@Transactional
public class AddressService {
    @Resource
    private AddressRepository addressRepository;
    
    public Address add(Address address) {
        return addressRepository.save(address);
    }
    
    public Address findById(Long id) {
        return addressRepository.findOne(id);
    }
}
```

b.UserService类

```
package spitter.service;

import javax.annotation.Resource;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import spittr.db.UserRepository;
import spittr.domain.User;

@Service
@Transactional
public class UserService {
    @Resource
    private UserRepository userRepository;
    
    public User addUser(User u) {
        return userRepository.save(u);
    }
    
    public User findByUsername(String username) {
        return userRepository.findByUserName(username);
    }
}
```

c.保存数据库并访问

```
package spitter.main;

import java.text.DateFormat;
import java.util.Date;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import spitter.config.JpaConfig;
import spitter.service.AddressService;
import spitter.service.UserService;
import spittr.domain.Address;
import spittr.domain.User;

public class Main {
    
    public static void main(String[] args) {
        ApplicationContext applicationContext=new AnnotationConfigApplicationContext(JpaConfig.class);
        
        UserService userService=applicationContext.getBean(UserService.class);
        
        AddressService addressService=applicationContext.getBean(AddressService.class);
        
        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);        
        String formattedDate = dateFormat.format(date);
        
        User user=new User();
        user.setUserName("test");
        user.setRegTime(formattedDate);
        user.setNickName("t");
        user.setPassWord("xxxxx");
        user.setEmail("test@qq.com");
        
        userService.addUser(user);//先注册用户完成
        
        //用户填写地址1
        Address address=new Address(null, "北京海淀区", "10000", "北京",user);
        
        addressService.add(address);
        
        Long addressId=address.getId();
        
        //用户填写地址2
        Address address2=new Address(null, "北京海淀区2", "10002", "北京2",user);
        
        addressService.add(address2);
        
        //从数据库中加载用户
        User dbUser=userService.findByUsername("test");
        
        //assert 2==dbUser.getAddressList().size();
        System.out.println(dbUser.getAddressList().size());
        
        //获取地址，并判断关联用户
        Address add=addressService.findById(addressId);
        
        //assert "test".equals(add.getUser().getUserName());
        System.out.println("test".equals(add.getUser().getUserName()));
    }
}
```

错误信息

```
Exception in thread "main" org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: spittr.domain.User.addressList, no session or session was closed
```

d.懒加载问题

可通过在User类的Address集合中设置EAGER加载方式解决。

```
@OneToMany(cascade=CascadeType.REMOVE,fetch=FetchType.EAGER)
@JoinColumn(name="user_id")
private List<Address> addressList;
```