---
title: spring cloud 服务提供者整合mybatis
tags: [spring]
---

1）查看application.yml中关于mybatis的配置信息

```
mybatis:
 config-location: classpath:mybatis/mybatis.cfg.xml # mybatis配置文件所在路径
 type-aliases-package: org.sjq.test.entities            # 所有Entity别名类所在包
 mapper-locations: classpath:mybatis/mapper/**/*.xml    # mapper映射文件
```

type-aliases-package会将包下的实体类作为mybatis映射文件的resultType和parameterType别名来使用。

mapper-locations指定的映射文件与@Mapper注解修饰dao层接口对应，生成相应的接口方法的实现。映射文件与dao层接口类的对应是通过namespace属性实现，sql的id与接口方法对应

2）创建mybatis配置文件mybatis.cfg.xml

在main/resources目录下创建mybatis，然后创建文件mybatis.cfg.xml

```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
    "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- mybatis的配置都可以在spring配置文件中进行配置 -->
    <settings>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

3）创建dao层类（接口）

```
package org.sjq.test.dao;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;
import org.sjq.test.entities.Dept;

//mybatis会自动扫描到该注解
@Mapper
public interface DeptDao {
    public boolean addDept(Dept dept);
    
    public Dept findById(Long id);
    
    public List<Dept> findAll();
}
```

注：不需要提供实现类，mybatis扫描到该@Mapper会根据接口方法名，到相应的mapper映射文件中匹配。

4）创建mapper映射文件

匹配规则：mapper映射文件的namespace值与接口的全限定名相同。对应的sql标签的id与方法名匹配。

在main/resources/mybatis目录下创建mapper目录，然后创建DeptMapper.xml，这里的文件名最好与实体bean的类名匹配，方便识别。

```
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.sjq.test.dao.DeptDao">

    <select id="findById" resultType="Dept" parameterType="Long">
        select deptno,dname,db_source from dept where deptno=#{deptNo}
    </select>
    
    <select id="findAll" resultType="Dept">
        select deptno,dname,db_source from dept
    </select>
    
    <insert id="addDept" parameterType="Dept">
        insert into dept(dname,db_source) values(#{dName},database());
    </insert>
</mapper>
```

注：xml中的resultType和parameterType与实体类的类名相同，是在什么地方定义的这种类型呢？答案是在application.yml中配置的type-aliases-package属性，该指定了实体类所在的包路径。会将该路径下的所有实体类的类名作为相应实体的别名，并应用到mapper映射文件xml中。

5）创建sql

```
drop database if exists cloudDB01;
create database cloudDB01 character set UTF8;
use cloudDB01;

create table dept(
    deptno bigint not null primary key auto_increment,
    dname varchar(60),
    db_source varchar(60)
);

insert into dept(dname,db_source) values('开发部',database());
insert into dept(dname,db_source) values('人事部',database());
insert into dept(dname,db_source) values('财务部',database());
insert into dept(dname,db_source) values('市场部',database());
insert into dept(dname,db_source) values('运维部',database());
```