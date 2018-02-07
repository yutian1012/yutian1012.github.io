---
title: JDK的SPI（Service Provider Interface）原理分析
tags: [basic]
---

SPI机制提供了一个表达接口和其具体实现类之间的绑定关系的方案。具体是在JAR包的"META-INF/services/"目录下建立一个文件，文件名是接口的全限定名，文件的内容可以有多行，每行都是该接口对应的具体实现类的全限定名。

参考：http://www.cnblogs.com/javaee6/p/3714719.html

1）SPI机制简介

SPI的全名为Service Provider Interface.大多数开发人员可能不熟悉，因为这个是针对厂商或者插件的。在java.util.ServiceLoader的文档里有比较详细的介绍。简单的总结下java spi机制的思想。我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块的方案，xml解析模块、jdbc模块的方案等。面向的对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。 java spi就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。

2）SPI具体约定

spi的具体约定为:当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。 基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。jdk提供服务实现查找的一个工具类：java.util.ServiceLoader。

3）应用场景

common-logging

apache最早提供的日志的门面接口。只有接口，没有实现。具体方案由各提供商实现， 发现日志提供商是通过扫描 META-INF/services/org.apache.commons.logging.LogFactory配置文件，通过读取该文件的内容找到日志提工商实现类。只要我们的日志实现里包含了这个文件，并在文件里制定 LogFactory工厂接口的实现类即可。

jdbc

jdbc4.0以前，开发人员还需要基于Class.forName的方式来装载驱动，jdbc4也基于spi的机制来发现驱动提供商了，可以通过META-INF/services/java.sql.Driver文件里指定实现类的方式来暴露驱动提供者。

### 案例说明

一个内容管理系统有一个搜索模块。是基于接口编程的。搜索的实现可能是基于文件系统的搜索，也可能是基于数据库的搜索

1）定义接口

```
package my.xyz.spi;  
import java.util.List;  
public interface Search {  
   public List serch(String keyword);  
}
```

2）不同厂商实现不同策略

A公司采用文件系统搜索的方式实现了Search接口，A公司实现的类com.A.spi.impl.FileSearch。

B公司采用了数据库系统的方式实现了Search接口，B公司实现的类com.B.spi.impl.DatabaseSearch。

那么A公司发布 实现jar包时，则要在jar包中META-INF/services/my.xyz.spi.Search文件中写下如下内容

```
com.A.spi.impl.FileSearch
```

那么B公司发布 实现jar包时，则要在jar包中META-INF/services/my.xyz.spi.Search文件中写下如下内容

```
com.B.spi.impl.DatabaseSearch
```

注：文件内容指明了接口实现类，而文件名则为接口类全名

3）在接口中提供ServiceLoader查询机制

接口中提供查询机制，能够让用户程序不需要实际导入实现类，就能成功的运行。

```
package com.xyz.factory;  
import java.util.Iterator;  
import java.util.ServiceLoader;  
import my.xyz.spi.Search;  
public class SearchFactory {  
    private SearchFactory() {  
    }
    public static Search newSearch() {  
        Search search = null;  
        ServiceLoader<Search> serviceLoader = ServiceLoader.load(Search.class);  
        Iterator<Search> searchs = serviceLoader.iterator();  
        if (searchs.hasNext()) {  
            search = searchs.next();  
        }  
        return search;  
    }  
}
```

4）使用

将A公司或B公司的实现jar包放置到类路径下，然后运行测试

```
package my.xyz.test;  
import java.util.Iterator;  
import java.util.ServiceLoader;  
import com.xyz.factory.SearchFactory;  
import my.xyz.spi.Search;  
public class SearchTest {  
    public static void main(String[] args) {  
        Search search = SearchFactory.newSearch();  
        search.serch("java spi test");  
    }  
} 
```