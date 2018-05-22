---
title: spring data jpa介绍和引入
tags: [spring]
---

帮助文档：https://docs.spring.io/spring-data/jpa/docs/current/reference/html/

Spring Data简化持久化对象的访问，声明JPARepository接口，让Spring Data JPA在运行时自动生成这些接口的实现。当需要的Repository方法超出了Spring Data JPA所提供的功能时，可以借助@Query注解以及以Impl为后缀的实现类来解决。

1）样板式代码

针对领域模型对象的查询，spring-jpa的查询一般也都是样板式的代码，依然还会直接与EntityManager交互来查询数据库。

```
public void addSpitter(Spitter spitter){ 
    entityManager.persist(spitter); 
}
```

注：在访问数据时也需要通过entityManager实现访问，针对不同的领域对象，都有相似的代码，这部分代码没有什么特别的逻辑，而且针对这种代码会在多处重复编写。

2）spring data的引入

a.Spring Data JPA能够终结这种样板式的愚蠢行为。

我们不再需要一遍遍地编写相同的Repository实现，Spring-Data能够让我们只编写Repository接口就可以了，根本就不再需要实现类了。

```
public interface SpitterRepository extends JpaRepository< Spitter, Long>{
}
```

注：repository接口只需要基础JpaRepository接口即可。

b.spring data提供了共性的方法

repository接口继承了JpaRepository即可后，会自动继承18个执行持久化操作的通用方法，如保存、删除以及根据ID查询相应领域对象的方法。

c.通过接口方法命名规则自动底层实现相关功能

3）原理分析

当spring data在指定的包路径下查找到继承了Repository接口的类后，会自动创建相应的实现类，同时实现接口中定义的方法实现。

Spring Data会检查Repository接口的所有方法，解析方法的名称，并基于被持久化的对象来试图推测方法的目的。本质上，Spring Data定义了一组小型的领域特定语言（domain-specific language，DSL），在这里，持久化的细节都是通过Repository方法的签名来描述的。