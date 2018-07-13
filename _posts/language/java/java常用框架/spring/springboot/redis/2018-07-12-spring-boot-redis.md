---
title: spring boot redis入门
tags: [spring]
---

参考：https://www.cnblogs.com/floay/p/6485742.html

1）系统要先安装并启动redis

redis安装与启动。

2）引入相关的依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

3）在application.yml文件中添加配置

```
# redis config
spring: 
  redis:
    host: 127.0.0.1
    password: 123456
    port: 6379
    timeout: 3000

# redis connection pool
spring:
  redis:
    pool:
      max-idle: 500
      min-idle: 50
      max-active: 2000
      max-wait: 1000
```

4）新建模型类（用于缓存的数据）

```
package org.ipph.spider.entity;

import java.util.Date;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.experimental.Accessors;

@Setter
@Getter
@NoArgsConstructor
@Accessors(chain=true)
public class PageLog extends PageModel{
    private String id;
    private String url;
    private String note;
    private Date createDate;
}
```

5）创建redis的service层

spring整合redis时，提供了StringRedisTemplate和RedisTemplate两个常用的template模板类，这两个类用于简化redis的访问。

```
package org.ipph.spider.redis;

import javax.annotation.Resource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

public class RedisService {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Resource(name = "stringRedisTemplate")
    private ValueOperations<String, String> valOpsStr;

    @Autowired
    private RedisTemplate<Object, Object> redisTemplate;

    @Resource(name = "redisTemplate")
    private ValueOperations<Object, Object> valOpsObj;

    /**
     * 根据指定key获取String
     * @param key
     * @return
     */
    public String getStr(String key){
        return valOpsStr.get(key);
    }

    /**
     * 设置Str缓存
     * @param key
     * @param val
     */
    public void setStr(String key, String val){
        valOpsStr.set(key,val);
    }

    /**
     * 删除指定key
     * @param key
     */
    public void del(String key){
        stringRedisTemplate.delete(key);
    }

    /**
     * 根据指定o获取Object
     * @param o
     * @return
     */
    public Object getObj(Object o){
        return valOpsObj.get(o);
    }

    /**
     * 设置obj缓存
     * @param o1
     * @param o2
     */
    public void setObj(Object o1, Object o2){
        valOpsObj.set(o1, o2);
    }

    /**
     * 删除Obj缓存
     * @param o
     */
    public void delObj(Object o){
        redisTemplate.delete(o);
    }
}
```

6）测试

