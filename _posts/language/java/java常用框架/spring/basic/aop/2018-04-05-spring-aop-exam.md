---
title: AOP的应用
tags: [spring]
---

实例：吟游诗人用诗歌记载骑士的事迹，并将其进行传唱。假设我们需要使用吟游诗人这个服务类来记载骑士的所有事迹。

1）吟游诗人类

该类相当于一个日志服务模块，用来记录骑士相关的信息。

```
public class Minstrel {

  private PrintStream stream;
  
  public Minstrel(PrintStream stream) {
    this.stream = stream;
  }
  
  //骑士探险前调用
  public void singBeforeQuest() {
    stream.println("Fa la la, the knight is so brave!");
  }

  //骑士探险后调用
  public void singAfterQuest() {
    stream.println("Tee hee hee, the brave knight " +
            "did embark on a quest!");
  }

}
```

2）骑士类

该类相当于业务实现类，即具体的业务逻辑

```
public class BraveKnight implements Knight {

  private Quest quest;
  //代码模块间出现了不相关的依赖
  private Minstrel minstrel;

  public BraveKnight(Quest quest,Minstrel minstrel) {
    this.quest = quest;
    this.minstrel=minstrel;
  }

  public void embarkOnQuest() {
    minstrel.singBeforeQuest();
    quest.embark();
    minstrel.singAfterQuest();
  }

}
```

思考：这样的代码合理吗？

为什么骑士还需要提醒吟游诗人去做他自己份内的事情。此外，因为骑士需要知道吟游诗人，所以必须把吟游诗人注入到Braveknight类中。另外，如果Minstrel对象空，是不是也需要考虑空指针的问题？因此，还需要应对没有吟游诗人的情况。而这些情况是该骑士去关心的吗？

3）使用AOP简化

将吟游诗人类配置成AOP切面，即一个包裹骑士类的外壳。而骑士类也从复杂的逻辑中解脱出来，不在依赖和关心Minstrel类。

```
public class BraveKnight implements Knight {

  private Quest quest;

  public BraveKnight(Quest quest) {
    this.quest = quest;
  }

  public void embarkOnQuest() {
    quest.embark();
  }

}
```

4）xml方式配置AOP

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop 
      http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="sia.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <bean id="minstrel" class="sia.knights.Minstrel">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <aop:config>
    <!-- 吟游诗人作为一个切面 -->
    <aop:aspect ref="minstrel">
      <!-- 定义何时需要吟游诗人出场，即什么时候调用
      这里原来骑士类的embarkOnQuest方法中要主动去提醒吟游诗人 -->
      <aop:pointcut id="embark"
          expression="execution(* *.embarkOnQuest(..))"/>
      
      <!-- 方法开始前调用切面的（吟游诗人）singBeforeQuest方法 -->
      <aop:before pointcut-ref="embark" 
          method="singBeforeQuest"/>
      
      <!-- 方法结束前调用切面的（吟游诗人）singAfterQuest方法 -->
      <aop:after pointcut-ref="embark" 
          method="singAfterQuest"/>
    </aop:aspect>
  </aop:config>
  
</beans>
```

5）理解切面，切点以及通知

a.切面

理解为对业务的包裹，相当于一层外壳，而要调用核心的业务必须要先通过这层外壳。外壳可以理解为调用核心前的服务，即调用前要执行的。

```
<aop:aspect ref="minstrel">
```

注：使用aop:aspect关键字来定义。

b.切点

即切入点，即什么地方（一般为业务的某个方法处）要调用切面，通过相关的表达式来定义切点。

```
<aop:pointcut id="embark" expression="execution(* *.embarkOnQuest(..))"/>
```

c.通知

通知即针对切点处的业务逻辑调用，要经过切面的哪个位置，即切面的哪个方法会被调用。另外通知按照调用时机分为调用前通知和调用后通知等。

```
<aop:before pointcut-ref="embark" method="singBeforeQuest"/>
```

6）优点

首先，Minstrel仍然是一个POJO，没有任何代码表明它要被作为一个切面使用。其次，Minstrel可以应用到BraveKnight中，而BraveKnight不需要显示的调用它。实际上，BraveKnight完全不知道minstrel的存在。