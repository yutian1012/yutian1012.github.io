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