---
title: spring条件实例化
tags: [spring]
---

但是Spring4引入了一个新的@Conditional注解。如果给定的条件计算结果为true，就会创建这个bean，否则的话，这个bean会被忽略。

1）实例

假设有一个名为MagicBean的类，我们希望只有设置了magic环境属性的时候，Spring才会实例化这个类。如果环境中没有这个属性，那么MagicBean将会被忽略。

```
public class MagicBean {

}

@Configuration
public class MagicConfig {

  @Bean
  @Conditional(MagicExistsCondition.class)
  public MagicBean magicBean() {
    return new MagicBean();
  }
}
```

注：需要在@Conditional注解中指定相关的条件类

注2：所有@Conditional注解上引用的bean都会被创建，不需要手动的创建该bean实例

2）实现条件类

```
package com.habuma.restfun;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class MagicExistsCondition implements Condition {

  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    Environment env = context.getEnvironment();
    return env.containsProperty("magic");
  }
  
}
```

注：从环境中获取magic参数，如果环境中设置了该属性，则返回true。

3）说明

条件判断类要实现Condition接口，spring在判断时会调用该接口定义的matches方法判断是否满足条件，满足则返回true，则相应的bean就会被创建。

```
public interface Condition {

    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

4）测试

```
import static org.junit.Assert.*;

import org.junit.BeforeClass;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=MagicConfig.class)
public class MagicExistsTest {

  @Autowired
  private ApplicationContext context;
  
  /*
   * This test will fail until you set a "magic" property.
   * You can set this property as an environment variable, a JVM system property, by adding a @BeforeClass
   * method and calling System.setProperty() or one of several other options.
   */
  @Test
  public void shouldNotBeNull() {
    assertTrue(context.containsBean("magicBean"));
  }
  
  @BeforeClass
  public static void setProperty() {
      System.setProperty("magic", "sss");
  }
  
}
```