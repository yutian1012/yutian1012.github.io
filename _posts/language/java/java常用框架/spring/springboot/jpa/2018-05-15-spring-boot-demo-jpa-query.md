---
title: spring boot data使用@Query查询
tags: [spring]
---

这种查询可以声明在Repository方法中，摆脱像命名查询那样的约束，将查询直接在相应的接口方法中声明，结构更为清晰，这是Spring data的特有实现。 

1）实例代码

```
public interface UserRepository extends JpaRepository<User, Long> { 

  @Query("select u from User u where u.emailAddress = ?1") 
  User findByEmailAddress(String emailAddress); 
} 
```

注：索引参数如下所示，索引值从1开始，查询中”?X”个数需要与方法定义的参数个数相一致，并且顺序也要一致。