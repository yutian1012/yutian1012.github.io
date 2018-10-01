---
title: spring容器的初始化实例
tags: [spring]
---

DefaultListableBeanFactory类是真正第一个可以独立的IOC容器，而且后面的ApplicationContext据说也是依据它来建立。

要分析spring容器的初始化操作最好在该类的基础上进行查看。

1）使用DefaultListableBeanFactory初始化容器

```
package soundsystem;

import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
import org.springframework.core.io.ClassPathResource;

public class DefaultListableBeanFactoryMain {
    public static void main(String[] args) {
        ClassPathResource res=new ClassPathResource("META-INF/spring/app-context.xml");
        DefaultListableBeanFactory factory=new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader=new XmlBeanDefinitionReader(factory);
        reader.loadBeanDefinitions(res);
        
        CompactDisc disc=factory.getBean(CompactDisc.class);
        disc.play();
    }
}
```

2）创建配置文件

在路径src/main/resources/META-INF/spring/下创建app-context.xml文件。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- <context:component-scan base-package="soundsystem" /> -->
    <bean id="SgtPeppers" class="soundsystem.SgtPeppers"></bean>
</beans>
```

3）相关类创建

```
//接口
package soundsystem;

public interface CompactDisc {
  void play();
}

//实现类
package soundsystem;

public class SgtPeppers implements CompactDisc {
  private String title = "Sgt. Pepper's Lonely Hearts Club Band";  
  private String artist = "The Beatles";
  
  public void play() {
    System.out.println("Playing " + title + " by " + artist);
  }
}
```