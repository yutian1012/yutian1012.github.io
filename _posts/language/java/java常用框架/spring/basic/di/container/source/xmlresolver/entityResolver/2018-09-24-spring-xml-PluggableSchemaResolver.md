---
title: spring容器的PluggableSchemaEntityResolver解析器
tags: [spring]
---

EntityResolver的作用就是为应用本身提供一个如何寻找DTD或Schema声明的方法，这里以PluggableSchemaEntityResolver来分析如何从http远程范围schema变换到查找jar文件中的schema。

配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="SgtPeppers" class="com.sjq.bean.container.beanfactory.bean.SgtPeppers"></bean>
</beans>
```

1）DelegatingEntityResolver类的resolveEntity方法

```
@Override
public InputSource resolveEntity(String publicId, String systemId) throws SAXException, IOException {
    if (systemId != null) {
        if (systemId.endsWith(DTD_SUFFIX)) {
            return this.dtdResolver.resolveEntity(publicId, systemId);
        }
        else if (systemId.endsWith(XSD_SUFFIX)) {
            return this.schemaResolver.resolveEntity(publicId, systemId);
        }
    }
    return null;
}
```

配置文件使用的是schema方式校验xml文档，因此会使用schemaResolver来处理（即PluggableSchemaEntityResolver）。

2）查看PluggableSchemaEntityResolver类的resolveEntity方法

```
@Override
public InputSource resolveEntity(String publicId, String systemId) throws IOException {
    if (systemId != null) {
        //这里的systemId是以http开头的uri地址，通过schemaMappings映射成了本的文档
        String resourceLocation = getSchemaMappings().get(systemId);
        if (resourceLocation != null) {
            Resource resource = new ClassPathResource(resourceLocation, this.classLoader);
            try {
                InputSource source = new InputSource(resource.getInputStream());
                source.setPublicId(publicId);
                source.setSystemId(systemId);
               
                return source;
            }
            catch (FileNotFoundException ex) {
                ...
            }
        }
    }
    return null;
}
```

查看getScheMappings方法

```
/**
 * Load the specified schema mappings lazily.
 */
private Map<String, String> getSchemaMappings() {
    if (this.schemaMappings == null) {
        synchronized (this) {
            if (this.schemaMappings == null) {
                
                try {
                    //从META-INF/spring.schemas文件中加载映射
                    Properties mappings =
                            PropertiesLoaderUtils.loadAllProperties(this.schemaMappingsLocation, this.classLoader);

                    //设置到映射集合中
                    Map<String, String> schemaMappings = new ConcurrentHashMap<String, String>(mappings.size());
                    CollectionUtils.mergePropertiesIntoMap(mappings, schemaMappings);
                    this.schemaMappings = schemaMappings;
                }
                catch (IOException ex) {
                    ...
                }
            }
        }
    }
    return this.schemaMappings;
}
```

查看spring-beans-xxx.jar包中的META-INF/spring.schemas文件

![](/images/spring/core/source/xmlresolver/spring-schema.png)

```
http\://www.springframework.org/schema/beans/spring-beans-2.0.xsd=org/springframework/beans/factory/xml/spring-beans-2.0.xsd
http\://www.springframework.org/schema/beans/spring-beans-2.5.xsd=org/springframework/beans/factory/xml/spring-beans-2.5.xsd
http\://www.springframework.org/schema/beans/spring-beans-3.0.xsd=org/springframework/beans/factory/xml/spring-beans-3.0.xsd
http\://www.springframework.org/schema/beans/spring-beans-3.1.xsd=org/springframework/beans/factory/xml/spring-beans-3.1.xsd
http\://www.springframework.org/schema/beans/spring-beans-3.2.xsd=org/springframework/beans/factory/xml/spring-beans-3.2.xsd
...
```