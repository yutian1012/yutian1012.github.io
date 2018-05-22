---
title: spring data jpa入门实例-xml方式配置
tags: [spring]
---

源码地址：https://github.com/yutian1012/springdemo/tree/master/jpademo

1）引入依赖

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>${spring.data.jpa.Version}</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>${org.hibernate.version}</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
</dependency>
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>${dbcp.version}</version>
</dependency>
```

注：version信息空的依赖都是从父项目中获取的

2）xml配置文件

在resources目录下创建applicationContext.xml。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa-1.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">


    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:app.properties</value>
            </list>
        </property>
    </bean>

    <jpa:repositories base-package="spittr.db" />
    
    <bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
        p:dataSource-ref="dataSource"
        p:persistenceUnitName="spitter"
        p:jpaVendorAdapter-ref="jpaVendorAdapter" 
        p:packagesToScan="spittr.domain"/>

    <bean id="jpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
        <property name="database" value="MYSQL" />
        <property name="showSql" value="true" />
        <property name="generateDdl" value="true" />
    </bean>

    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager"
        p:entityManagerFactory-ref="emf" />
        
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"  destroy-method="close">
        <property name="driverClassName" value="${driverClassName}"></property>
        <property name="url" value="${url}"></property>
        <property name="username" value="${username}"></property>
        <property name="password" value="${password}"></property>
    </bean>
</beans>
```

创建属性文件app.properties

```
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
username=root
password=root
```

3）分析标签

a.jpa:repositories元素

该元素掌握了SpringDataJPA的所有魔力，该元素会扫描它的基础包来查找扩展自Spring Data JPA Repository接口的所有接口。如果发现了扩展自Repository的接口，它会自动生成（在应用启动的时候）这个接口的实现。

4）创建实体类

```
@Entity
public class Spitter {
  
  private Spitter() {}
  @Id
  @GeneratedValue(strategy=GenerationType.IDENTITY)
  private Long id;
  @Column(name="username")
  private String username;
  @Column(name="password")
  private String password;
  @Column(name="fullname")
  private String fullName;
  @Column(name="email")
  private String email;
  @Column(name="updateByEmail")
  private boolean updateByEmail;
  @Column(name="status")
  private String status;
  //省略set和get方法
}
```

5）创建dao方法

```
package spittr.db;
import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;
import spittr.domain.Spitter;

public interface SpitterRepository extends JpaRepository<Spitter, Long> {
    Spitter findByUsername(String username);   
    List<Spitter> findByUsernameOrFullNameLike(String username, String fullName);
}
```

6）测试

```
package spittr.db;

import static org.junit.Assert.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import spittr.domain.Spitter;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:applicationContext.xml")
public class SpitterRepositoryTest {

    @Autowired
    SpitterRepository spitterRepository;
    
    @Test
    @Transactional
    public void testSave() {
        Spitter spitter = new Spitter(null, "newbee", "letmein", "New Bee", "newbee@habuma.com", true);
        spitterRepository.save(spitter);
        assertEquals(1, spitterRepository.count());
    }

}
```