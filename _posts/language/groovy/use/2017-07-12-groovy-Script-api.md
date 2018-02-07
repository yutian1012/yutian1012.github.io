---
title: 将groovy整合到应用中--java调用脚本API
tags: [groovy]
---

参考：http://www.groovy-lang.org/integrating.html

Java SE 6 引入了对 Java Specification Request（JSR）223 的支持，JSR 223旨在定义一个统一的规范，使得Java应用程序可以通过一套固定的接口与各种脚本引擎交互，从而达到在 Java 平台上调用各种脚本语言的目的。每一个脚本引擎就是一个脚本解释器，负责运行脚本，获取运行结果。ScriptEngine 接口提供了许多 eval 函数的变体用来运行脚本，这个函数的功能就是获取脚本输入，运行脚本，最后返回输出。

### 实例

1）测试实例执行脚本

```
@Test
public void testGroovyApi(){
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("groovy");
    Integer sum;
    try {
        sum = (Integer) engine.eval("(1..10).sum()");
        assert 55== sum;
    } catch (ScriptException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
}
```

2）传递数据

```
@Test
public void testGroovyApiShareData(){
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("groovy");
    try {
        engine.put("first", "HELLO");
        engine.put("second", "world");
        String result = (String) engine.eval("first.toLowerCase() + ' ' + second.toUpperCase()");
        assert "hello WORLD".equals(result);
    } catch (ScriptException e) {
        e.printStackTrace();
    }
}
```

3）使用Invocable执行脚本方法

```
@Test
public void testGrrovyApiInvoke(){
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("groovy");
    String fact = "def factorial(n) { n == 1 ? 1 : n * factorial(n - 1) }";
    try {
        engine.eval(fact);//初始化构建执行脚本，如何使println "xxx"，会直接输出
        Invocable inv = (Invocable) engine;
        Object[] params = {5};//传递参数
        Object result = inv.invokeFunction("factorial", params);//调用方法
        assert 120 == (int)result;
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

注：调用方法

### Scripting的API接口

1）ScriptEngineManager类

它是使用Scripting API的入口，ScriptEngineManager可以通过jar服务发现(service discovery)机制寻找合适的脚本引擎类(ScriptEngine)，使用Scripting API的最简单方式只需下面三步

1、创建一个ScriptEngineManager对象

2、通过ScriptEngineManager获得ScriptEngine对象

3、用ScriptEngine的eval方法执行脚本

```
@Test
public void testGroovyApiUse() throws ScriptException{
    ScriptEngineManager factory = new ScriptEngineManager();//step1
    ScriptEngine engine = factory.getEngineByName("JavaScript");//Step2
    engine.eval("print('Hello, Scripting')");//Step3
}
```

注：这里执行的是Javascript脚本，使用的是Javascript脚本引擎

2）ScriptEngine类

脚本引擎类，通过脚本引擎可以执行相应的脚本代码。

3）ScriptContext 接口

提供脚本运行上下文环境，即脚本中的变量存取设置环境

```
@Test
public void testGroovyApiContext() throws Exception{
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("groovy");   
    ScriptContext context=engine.getContext();
    context.setAttribute("script", "Groovy",ScriptContext.ENGINE_SCOPE);
    
    engine.eval("println 'Hello, '+script");
}
```

注：可以设置输出环境，绑定变量setAttribute，甚至可以设置Binding对象绑定参数。

4）Bindings 接口

利用Bindings接口实现变量存取

```
@Test
public void testGroovyApiBinding() throws Exception{
    ScriptEngineManager factory = new ScriptEngineManager();
    ScriptEngine engine = factory.getEngineByName("groovy");
    
    //设置参数
    Bindings binding=new SimpleBindings();
    binding.put("script", "Groovy");
    
    engine.setBindings(binding, ScriptContext.ENGINE_SCOPE);
    
    engine.eval("println 'Hello, '+script");
}
```

5）Invocable 接口

允许 Java 平台调用脚本程序中的函数或方法。Invocable 接口还允许 Java 应用程序从这些函数中直接返回一个接口，通过这个接口实例来调用脚本中的函数或方法，从而我们可以从脚本中动态的生成 Java 应用中需要的接口对象。

定义接口

```
package groovyJava.integreate;

interface GreeterInterface {
    String sayHello();
}
```

实现测试

```
@Test
public void testGroovyApiInvocate() throws ScriptException, NoSuchMethodException{
    String script = 
            "String sayHello() {"+
                "String greet = 'Hello, world123!';"+
                "return greet;"+
            "}";//这是一个实现类
    
    ScriptEngineManager manager = new ScriptEngineManager();
    ScriptEngine engine = manager.getEngineByName("groovy");
    engine.eval(script);
    
    Invocable invocable = (Invocable) engine; 
    
    GreeterInterface greeter = invocable.getInterface(GreeterInterface.class);
    
    Object result = greeter.sayHello();
    System.out.println(result);  
}
```

注：这里只需要给出接口的实现方法即可，不需要一个完整的实现类

6）Compilable 接口

允许 Java 平台编译脚本程序，供多次调用

```
@Test
public void testGroovyApiCompiable() throws ScriptException{
    String script = "println greeting; greeting= 'Good Afternoon!' ";
    ScriptEngineManager manager = new ScriptEngineManager();
    ScriptEngine engine = manager.getEngineByName("groovy");
    engine.put("greeting", "Good Morning!");
   
    if (engine instanceof Compilable) {
        Compilable compilable = (Compilable) engine;
        CompiledScript compiledScript = compilable.compile(script);
        compiledScript.eval();
        compiledScript.eval();
    }
}
```

注：第一次运行设置变量greeting值为Good Morning!，运行结束后设置其值为Good Afternoon!，并且可以不需要重新编译继续运行。

### 存储属性级别

Script脚本API共有三个级别的地方可以存取属性，分别是ScriptEngineManager中的Bindings，ScriptEngine 实例对应的ScriptContext中含有的 Bindings，以及调用eval函数时传入的Bingdings。离函数调用越近，其作用域越小，优先级越高。

```
@Test
public void testGroovyApiAttribute() throws ScriptException{
    String script="println greeting";
    ScriptEngineManager manager = new ScriptEngineManager();
    ScriptEngine engine = manager.getEngineByName("groovy");
   
    //Attribute from ScriptEngineManager
    manager.put("greeting", "Hello from ScriptEngineManager");
    engine.eval(script);

    //Attribute from ScriptEngine
    engine.put("greeting", "Hello from ScriptEngine");
    engine.eval(script);

    //Attribute from eval method
    ScriptContext context = new SimpleScriptContext();
    context.setAttribute("greeting", "Hello from eval method", ScriptContext.ENGINE_SCOPE);
    engine.eval(script,context);
}
```