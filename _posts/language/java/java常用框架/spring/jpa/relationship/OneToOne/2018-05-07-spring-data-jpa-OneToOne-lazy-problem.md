---
title: Spring Date Jpa懒加载问题
tags: [spring]
---

不使用@Transaction时出现no sessiont，懒加载问题。

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