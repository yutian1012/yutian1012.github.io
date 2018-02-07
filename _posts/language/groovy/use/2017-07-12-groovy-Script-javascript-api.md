---
title: 将groovy整合到应用中--java调用javaScript脚本方法API
tags: [groovy]
---

JSR 223提供的脚本API功能可以实现java运行javascript脚本。支持这种方法调用方式的脚本引擎可以实现javax.script.Invocable接口。JavaSE自带的JavaScript脚本引擎实现了Invocable接口。


通过Invocable接口可以调用脚本中的顶层方法，也可以调用对象中的成员方法。如果脚本中顶层方法或对象中的成员方法实现了Java中的接口，可以通过Invocable接口中的方法来获取脚本中相应的Java接口的实现对象。这样就可以在Java语言中定义接口，在脚本中实现接口。程序中使用该接口的其他部分代码并不知道接口是由脚本来实现的。

参考：http://book.51cto.com/art/201205/339163.htm

1）java调用脚本顶层方法

```
@Test
public void invokeFunction() throws ScriptException, NoSuchMethodException {  
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("JavaScript");
    String scriptText = "function greet(name) { println('Hello, ' + name); } ";
    engine.eval(scriptText);  
    Invocable invocable = (Invocable) engine;  
    invocable.invokeFunction("greet", "Alex");  
}
```

2）调用脚本中对象的成员方法

```
@Test
public void invokeMethod() throws ScriptException, NoSuchMethodException {  
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("JavaScript");
    String scriptText = "var obj = { getGreeting : function(name) { return 'Hello, ' + name; } }; ";  
    engine.eval(scriptText);  
    Invocable invocable = (Invocable) engine;  
    Object scope = engine.get("obj");  
    Object result = invocable.invokeMethod(scope, "getGreeting", "AlexMethod");
    System.out.println(result);  
} 
```

3）调用实现接口

在有些脚本引擎中，可以在Java语言中定义接口，并在脚本中编写接口的实现。这样程序中的其他部分可以只同Java接口交互，并不需要关心接口是由什么方式来实现的。

定义Greet接口，其中包含一个getGreeting方法。

```
public interface Greet {
    String getGreeting();
}
```

在脚本中实现这个接口。通过getInterface方法可以得到由脚本实现的这个接口的对象，并调用其中的方法。脚本中实现接口，只需要脚本实现接口中的方法即可。

```
@Test
public void useInterface() throws ScriptException {  
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("JavaScript");
    String scriptText = "function getGreeting(name) { return 'Hello, ' + name; } ";
    engine.eval(scriptText);  
    Invocable invocable = (Invocable) engine;  
    Greet greet = invocable.getInterface(Greet.class);  
    System.out.println(greet.getGreeting("AlexInterface"));  
} 
```