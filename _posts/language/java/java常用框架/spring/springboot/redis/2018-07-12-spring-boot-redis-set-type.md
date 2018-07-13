---
title: spring boot redis set结构类型的操作
tags: [spring]
---

1）redis操作类

```
package org.ipph.spider.redis;

import java.util.Set;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

@Service
public class RedisService {
    
    public static final String PATENT_FEE_PAID_KEY="appNumber_num";//申请号_条数
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 判断是否存在
     * @param value
     * @return
     */
    public boolean isExists(String value) {
        return stringRedisTemplate.boundSetOps(RedisService.PATENT_FEE_PAID_KEY).isMember(value);
    }
    /**
     * 向set集合中添加value
     * @param value
     */
    public void add(String value) {
        stringRedisTemplate.boundSetOps(RedisService.PATENT_FEE_PAID_KEY).add(value);
    }
    /**
     * 添加集合操作
     * @param data
     */
    public void addAll(Set<String> data) {
        String[] putData=new String[data.size()];
        data.toArray(putData);
        stringRedisTemplate.boundSetOps(RedisService.PATENT_FEE_PAID_KEY).add(putData);
    }
    /**
     * 获取set集合对象
     * @return
     */
    public Set<String> getAllSet(){
        return stringRedisTemplate.boundSetOps(RedisService.PATENT_FEE_PAID_KEY).members();
    }
}
```

2）单元测试

```
package org.ipph.spider.redis;

import static org.junit.Assert.*;

import java.util.HashSet;
import java.util.Set;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisServiceTest {

    @Resource
    private RedisService redisService;
    
    @Test
    public void testRedisSet() {
        
        Set<String> set=new HashSet<>();
        set.add("name1");
        set.add("name2");
        
        redisService.addAll(set);
        
        //从redis中获取
        boolean exists=redisService.isExists("name1");
        assertTrue(exists);
        
        redisService.add("name3");
        
        Set<String> tmp=redisService.getAllSet();
        assertTrue(tmp.size()==3);
        
        System.out.println(tmp.toArray().toString());
    }

}
```

3）redis-cli连接查看

```
# 客户端连接
redis-cli.exe -h 127.0.0.1 -p 6379

# 授权认证
auth 123456

# 查看keys
keys *

# 查询set集合内容
smembers appNumber_num

# 清空数据
flushdb

# 退出
exit
```