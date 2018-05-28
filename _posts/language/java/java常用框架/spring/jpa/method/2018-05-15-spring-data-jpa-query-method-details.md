---
title: spring data Jpa设置自定查询方法
tags: [spring]
---

@Query主键实现查询的一些实例

1）通过传递集合参数实现查询

```
public interface SpitterRepository extends JpaRepository<Spitter, Long> ,SpitterExtentionRepository{
    ...
    //添加查询方法
    @Query("select p from Spitter as p where p.username in (?1)")
    List<Spitter> findSpitterListByUserName(List<String> username);
}
```

测试方法

```
@Test
@Transactional
public void testQueryList() {
    Spitter spitter = new Spitter(null, "newbee", "letmein", "New Bee", "newbee@habuma.com", true);
    spitterRepository.save(spitter);
    Spitter spitter2 = new Spitter(null, "newbee2", "letmein", "New Bee", "newbee@habuma.com", true);
    spitterRepository.save(spitter2);
    
    List<String> userNameList=new ArrayList<>();
    userNameList.add("newbee");
    userNameList.add("newbee2");
    
    List<Spitter> spitterList=spitterRepository.findSpitterListByUserName(userNameList);
    
    assertNotNull(spitterList);
    
    assertTrue(2==spitterList.size());
}
```

2）返回单独字段

有时候在查询数据时，并不需要返回对象的所有字段，只需要返回特定的字段。

```
public interface SpitterRepository extends JpaRepository<Spitter, Long> ,SpitterExtentionRepository{
   
   //只返回username字段
    @Query("select p.username from Spitter as p ")
    List<Object> findUserNameList();
}
```

注：无法封装成Spitter对象返回

测试类

```
@Test
@Transactional
public void testSelectSingleField() {
    Spitter spitter = new Spitter(null, "newbee", "letmein", "New Bee", "newbee@habuma.com", true);
    spitterRepository.save(spitter);
    Spitter spitter2 = new Spitter(null, "newbee2", "letmein2", "New Bee2", "newbee2@habuma.com", true);
    spitterRepository.save(spitter2);
    List<Object> result=spitterRepository.findUserNameList();
    assertNotNull(result);
    assertEquals(2, result.size());
}
```

3）返回多个字段

内部返回的是对象的字段所组成的Object数组

```
public interface SpitterRepository extends JpaRepository<Spitter, Long> ,SpitterExtentionRepository{
    //返回多个字段，字段不是用map封装，而是使用Object数组封装的
    @Query("select p.id,p.username from Spitter as p ")
    List<Object[]> findUserNameAndIdList();
}
```

测试类

```
@Test
@Transactional
public void testSelectFieldWithId() {
    Spitter spitter = new Spitter(null, "newbee", "letmein", "New Bee", "newbee@habuma.com", true);
    spitterRepository.save(spitter);
    Spitter spitter2 = new Spitter(null, "newbee2", "letmein2", "New Bee2", "newbee2@habuma.com", true);
    spitterRepository.save(spitter2);
    List<Object[]> result=spitterRepository.findUserNameAndIdList();
    assertNotNull(result);
    assertEquals(2, result.size());
    for(Object[] s:result) {
        System.out.println(s[1]);//输出username
    }
}
```