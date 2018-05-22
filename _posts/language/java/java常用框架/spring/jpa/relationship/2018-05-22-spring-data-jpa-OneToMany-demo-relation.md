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