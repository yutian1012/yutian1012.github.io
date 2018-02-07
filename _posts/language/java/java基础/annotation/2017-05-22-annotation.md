---
title: java注解
tags: [annotation]
---

参考：http://www.cnblogs.com/skywang12345/p/3344137.html

参考：http://www.cnblogs.com/peida/archive/2013/04/26/3038503.html

### 注解不需要指定变量名

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {

    /**
     * The value may indicate a suggestion for a logical component name,
     * to be turned into a Spring bean in case of an autodetected component.
     * @return the suggested component name, if any
     */
    String value() default "";

}
```
