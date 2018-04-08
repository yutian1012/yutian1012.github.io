---
title: spring的装配java配置方式
tags: [spring]
---

在进行显式配置的时候，有两种可选方案：Java和XML。

1）单独放置javaConfig配置类

使用注解@Configuration定义配置类。配置类不应该包含任何业务逻辑，JavaConfig也不应该侵入到业务逻辑代码之中。尽管不是必须的，但通常会将JavaConfig放到单独的包中，使它与其他的应用程序逻辑分离开来，这样对于它的意图就不会产生困惑了。

```
package soundsystem;
import org.springframework.context.annotation.Configuration;
@Configuration
public class JavaConfig{}
```

注：@Configuration注解表示该类是一个配置类。即该类定义了应用上下文如何创建装配bean的细节。

2）创建bean定义信息

使用注解@Bean来定义spring容器管理的bean对象。@Bean注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean。方法体中包含了最终产生bean实例的逻辑。

```
@Bean( name=" lonelyHeartsClubBand") 
public CompactDisc sgtPeppers() {
    return new SgtPeppers(); 
}
```

在创建bean使可以做一些处理，动态返回实现对象，规则可以任意指定

```
@Bean 
public CompactDisc randomBeatlesCD() { 
    int choice = (int) Math.floor( Math. random() * 4);
    if (choice == 0) { 
        return new SgtPeppers(); 
    } else if (choice == 1) {
        return new WhiteAlbum(); 
    } else if (choice == 2) { 
        return new HardDaysNight(); 
    } else { 
        return new Revolver(); 
    } 
}
```

3）java配置方式注入bean

a.调用相关的@Bean注解方法获取实例

```
@Bean 
public CDPlayer cdPlayer() { 
    return new CDPlayer( sgtPeppers()); 
}
```

注：调用sgtPeppers()方法上添加了@Bean注解，Spring将会拦截所有对它的调用，并确保直接返回该方法所创建的bean，而不是每次都对其进行实际的调用。从而也能保证类的单例。

b.实例对象的获取交由spring来完成

```
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc){
    return new CDPlayer(compactDisc);
}
```

在这里，cdPlayer()方法请求一个CompactDisc作为参数。当Spring调用cdPlayer()创建CDPlayer bean的时候，它会自动装配一个CompactDisc到配置方法之中。然后，方法体就可以按照合适的方式来使用它。借助这种技术，cdPlayer()方法也能够将CompactDisc注入到CDPlayer的构造器中，而且不用明确引用CompactDisc的@Bean方法。

注：这种方式不会要求调用的方法必须处于同一个java配置类中。

4）多个config类整合

使用@Import注解引入其他配置类

```
package soundsystem;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class CDConfig{
    @Bean
    public CompactDisc compactDisc(){
        returnnewSgtPeppers();
    }
}

package soundsystem;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
@Configuration
@Import(CDConfig.class)
public class CDPlayerConfig{
    @Bean
    public CDPlayer cdPlayer(CompactDisccompactDisc){
        return new CDPlayer(compactDisc);
    }
}
```

另外的处理方式

```
package soundsystem;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
@Configuration
@Import({CDPlayerConfig.class,CDConfig.class})
public class SoundSystemConfig{}
```