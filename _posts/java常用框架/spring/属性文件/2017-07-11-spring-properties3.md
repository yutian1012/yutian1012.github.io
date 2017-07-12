---
title: spring属性文件的自定义类获取--继承PropertyPlaceholderConfigurer
tags: [spring]
---

参考：http://www.cnblogs.com/hafiz/p/5876243.html

### 自定义类获取属性

自定义类继承PropertyPlaceholderConfigurer，在自定义类中扩展获取属性值方法。

1）新建自定义类

```
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.config.PropertyPlaceholderConfigurer;

import java.util.Properties;

public class PropertyConfigurer extends PropertyPlaceholderConfigurer {

    private Properties props;// 存取properties配置文件key-value结果

    @Override
    protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess, Properties props)throws BeansException {
        super.processProperties(beanFactoryToProcess, props);
        this.props = props;
    }

    public String getProperty(String key){
        return this.props.getProperty(key);
    }

    public String getProperty(String key, String defaultValue) {
        return this.props.getProperty(key, defaultValue);
    }

    public Object setProperty(String key, String value) {
        return this.props.setProperty(key, value);
    }
}
```

注：增加了一个成员变量Properties，用于设置和获取属性。

2）配置文件

```
<bean id="propertyConfigurer" class="com.hafiz.www.util.PropertyConfigurer">
   <property name="ignoreUnresolvablePlaceholders" value="true"/>
   <property name="ignoreResourceNotFound" value="true"/>
   <property name="locations">
       <list>
          <value>classpath:jdbc.properties</value>
       </list>
   </property>
</bean>
```

3）使用

在使用时，可将propertyConfigurer类注入到想要使用的类中的属性。

```
@Component
public class Test{
    @Resource
    private PropertyConfigurer propertyConfigurer;

    public void test(){
        String test=propertyConfigurer.getProperty("xxxx");
    }
}
```