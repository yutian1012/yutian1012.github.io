---
title: spring容器的初始化过程
tags: [spring]
---

参考：https://www.cnblogs.com/ToBeAProgrammer/p/5230065.html

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
        //对Resource文件进行定位
        ClassPathResource res=new ClassPathResource("META-INF/spring/app-context.xml");
        DefaultListableBeanFactory factory=new DefaultListableBeanFactory();

        //创建一个载入BeanDefinition的读取器，将bean定义信息设置到BeanFactory
        XmlBeanDefinitionReader reader=new XmlBeanDefinitionReader(factory);

        //从资源文件中加载BeanDefinition定义信息
        reader.loadBeanDefinitions(res);
        
        //从BeanFactory中获取bean对象，伴随实例的初始化
        CompactDisc disc=factory.getBean(CompactDisc.class);
        disc.play();
    }
}
```

注：查看reader.loadBeanDefinitions，该方法是查看容器加载bean配置的入口方法。

2）XmlBeanFactory类

XmlBeanFactory是DefaultListableBeanFactory的子类，该类的代码执行与上面的编写的代码是相同的。

注：不过该类已经是@Deprecated。

3）DefaultListableBeanFactory的初始化

DefaultListableBeanFactory对象初始化会设置一些默认的环境对象。

3）解析过程

具体xml文件的解析交由XmlBeanDefinitionReader完成解析，完成整个载入和注册Bean定义的过程。

4）实例化过程

通过BeanFactory的getBean方法，获取对应的Bean。获取bean实例过程中会涉及到了Bean的实例化以及依赖注入。


