---
title: zookeeper java 客户端连接
tags: [zookeeper]
---

新建spring boot项目

1）新建maven项目，并添加相关依赖

这里除了要引入zookeeper相关的类外，还需要引入spring-boot-starter以及测试jar包spring-boot-starter-test

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.sjq.test</groupId>
  <artifactId>zk</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
    <relativePath/>
  </parent>
  
  <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <fastjson.version>1.2.6</fastjson.version>
    </properties>
    
  <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.12</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

2）创建应用启动类

```
package org.sjq.test.zk;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ZKApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZKApplication.class, args);
    }
}
```

3）创建zookeeper访问类

```
package org.sjq.test.zk;

import java.io.IOException;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;
import org.springframework.stereotype.Component;

@Component
public class ZKService {
    
    private ZooKeeper zookeeper;
    
    public synchronized void init() throws IOException {
        if(zookeeper ==null) {
            zookeeper=new ZooKeeper("127.0.0.1:2181", 30000, null);
        }
    }
    /**
     * 创建节点
     * @param node
     * @param data
     * @return
     * @throws IOException
     * @throws InterruptedException 
     * @throws KeeperException 
     */
    public String createNode(String node,String data) throws Exception {
        if(this.zookeeper==null) {
            init();
        }
        return zookeeper.create(node, data.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }
    /**
     * @param node
     * @return
     */
    public boolean isNodeExists(String node)throws Exception {
        Stat stat=getNodeStat(node);
        if(stat==null) {
            return false;
        }
        return true;
    }
    /**
     * 获取stat对象
     * @param node
     * @return
     * @throws Exception
     */
    public Stat getNodeStat(String node) throws Exception{
        if(this.zookeeper==null) {
            init();
        }
        return zookeeper.exists(node, false);
    }
    
    /**
     * 获取节点上设置的数据
     * @param node
     * @return
     */
    public String getNodeDate(String node) throws Exception{
        Stat stat=getNodeStat(node);
        if(null!=stat) {
            byte[] result=zookeeper.getData(node, false, stat);
            return new String(result);
        }
        return null;
    }
}
```

4）创建测试类

```
package org.sjq.test.zk;

import static org.junit.Assert.*;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ZKServiceTest {
    @Resource
    private ZKService zkService;

    @Test
    public void testZK() throws Exception {
        String node="/zktest";
        
        if(!zkService.isNodeExists(node)) {
            zkService.createNode(node, "testnode");
        }
        
        String data=zkService.getNodeDate(node);
        
        assertTrue("testnode".equals(data));
    }

}
```

5）客户端查看

```
# 连接服务
zkCli.cmd -server 127.0.0.1:2181

# 查看测试类创建的节点
ls /
```