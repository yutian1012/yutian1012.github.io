---
title: Hashcode和equals的相关工具类
tags: [basic]
---

1）Guava项目

Google的Guava项目里有处理hashCode和equals的工具类Objects.

```
com.google.common.base.Objects
```

maven项目中引入依赖

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>23.0</version>
</dependency>
```

2）apache commons工具

Apache Commons也有类似的工具类EqualsBuilder和HashCodeBuilder

maven项目中引入依赖

```
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
```

具体使用

```
public int hashCode()  {
    return new HashCodeBuilder(17, 37)
       .append(firstName)
       .append(lastName).toHashCode( );
} 
public boolean equals(Object o)  {
    boolean equals = false ;
    if(o != null && People.class.isAssignableFrom(o)){
        People p=(People) o;
        equals = (new EqualsBuilder()
           .append(firstName, ps.firstName)
           .append(lastName, ps.lastName)).isEquals();
    } 
     return  equals;
}
```

3）java7提供的工具

```
java.util.Objects
```

注：jdk环境必须高于等于java7.

4）IDE工具生成

常用IDE都提供hashCode()和equals()的代码生成。