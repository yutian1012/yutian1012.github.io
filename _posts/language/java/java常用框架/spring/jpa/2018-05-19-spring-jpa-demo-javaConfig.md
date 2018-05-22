---
title: spring data jpa入门实例-java类方式配置
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

2）创建配置类

```
package spittr.db;

import javax.inject.Inject;
import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

import org.apache.commons.dbcp2.BasicDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableJpaRepositories(basePackages="spittr.db")
@PropertySource("classpath:app.properties")
public class JpaConfig {
    
  @Autowired
  Environment env;

  @Bean
  public DataSource dataSource() {
      BasicDataSource dataSource=new BasicDataSource();
      /*dataSource.setDriverClassName(env.getProperty("driverClassName"));
      dataSource.setUrl(env.getProperty("url"));
      dataSource.setUsername(env.getProperty("username"));//返回了计算机的用户名，在使用时需要注意
      dataSource.setPassword(env.getProperty("password"));*/
      
      dataSource.setDriverClassName(env.getProperty("db.driverClassName"));
      dataSource.setUrl(env.getProperty("db.url"));
      dataSource.setUsername(env.getProperty("db.username"));
      dataSource.setPassword(env.getProperty("db.password"));
      return dataSource;
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter) {
    LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
    emf.setDataSource(dataSource);
    emf.setPersistenceUnitName("spittr");
    emf.setJpaVendorAdapter(jpaVendorAdapter);
    emf.setPackagesToScan("spittr.domain");
    return emf;
  }
  
  @Bean
  public JpaVendorAdapter jpaVendorAdapter() {
    HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
    adapter.setDatabase(Database.MYSQL);
    adapter.setShowSql(true);
    adapter.setGenerateDdl(true);
    adapter.setDatabasePlatform("org.hibernate.dialect.MySQL5InnoDBDialect");
    return adapter;
  }
  //嵌套定义配置类
  @Configuration
  @EnableTransactionManagement
  public static class TransactionConfig {

    @Inject
    private EntityManagerFactory emf;

    @Bean
    public PlatformTransactionManager transactionManager() {
      JpaTransactionManager transactionManager = new JpaTransactionManager();
      transactionManager.setEntityManagerFactory(emf);
      return transactionManager;
    }    
  }
  
}
```

注：在使用Environment对象获取属性文件信息时需要小心，获取连接数据库的用户名时，不要使用username，会与计算机当前登录用户返回同样的值，造成无法正确连接数据库。因此，最好在定义数据库连接用户名时使用db.username

```
# 返回了计算机的用户名，在使用时需要注意
System.out.println(env.getProperty("username"));
# 能成功获取属性文件中定义的用户名
System.out.println(env.getProperty("db.username"));
```

创建属性文件

```
db.driverClassName=com.mysql.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
db.username=root
db.password=root
```

3）分析标签

a.@EnableJpaRepositories注解

该注解掌握了SpringDataJPA的所有魔力，该注解会扫描它的基础包来查找扩展自Spring Data JPA Repository接口的所有接口。如果发现了扩展自Repository的接口，它会自动生成（在应用启动的时候）这个接口的实现。

注：相当于xml配置文件中的jpa:repositories元素

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
@ContextConfiguration(classes=JpaConfig.class)
public class SpitterRepositoryConfigTest {

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