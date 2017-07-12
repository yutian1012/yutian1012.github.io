---
title: 将groovy整合到应用中--GroovyScriptEngine
tags: [groovy]
---

参考：http://www.cnblogs.com/jevo/archive/2013/04/08/2992445.html

### GroovyScriptEngine运行执行脚本

该功能类似GroovyShell的运行脚本方式

1）新建脚本文件SimpleScript.groovy

```
println "Welcome to $language"
```

2）创建测试类

```
@Test
void testScript(){
    GroovyScriptEngine engine=new GroovyScriptEngine("src/groovyJava/integreate/");//指定一个脚本文件夹中路径
    def binding = new Binding()
    binding.setVariable("language", "HelloGroovyScriptEngine");
    def greeter = engine.run('SimpleScript.groovy', binding)
}
```

注：在创建GroovyScriptEngine时需要指定加载脚本的路径，运行run命令时指定脚本文件名。

### GroovyScriptEngine运行执行加载类

1）新建类文件ReloadingTest.groovy

```
class Greeter {
    String sayHello() {
        def greet = "Hello, world!"
        greet
    }
}

new Greeter()
```

注：在创建时不要使用new--groovy class方式创建，而应该使用new--file方式创建文件。为了防止歧义，可将ReloadingTest.groovy重命名为ReloadingTest.txt

创建ReloadingTest.txt

```
class Greeter {
    String sayHello() {
        def greet = "Hello, world!"
        greet
    }
}

new Greeter()
```

2）创建测试类

```
/**
 * 通过加载类，并运行
 */
@Test
void testClass(){
    GroovyScriptEngine engine=new GroovyScriptEngine("src/groovyJava/integreate/");//指定一个脚本文件夹中路径
    def binding = new Binding()
    //def greeter=engine.run("ReloadingTest.groovy", binding);
    def greeter=engine.run("ReloadingTest.txt", binding);
    println greeter.sayHello()
}
```

### GroovyScriptEngine的动态加载

在运行脚本时，GroovyScriptEngine是动态加载执行脚本的。

1）修改测试代码

```
/**
 * 通过加载类，并运行
 */
@Test
void testReloadClass(){
    GroovyScriptEngine engine=new GroovyScriptEngine("src/groovyJava/integreate/");//指定一个脚本文件夹中路径
    def binding = new Binding()
    while(true){
        def greeter=engine.run("ReloadingTest.txt", binding);
        println greeter.sayHello()
        Thread.sleep(1000);//暂停
    }
}
```

注：利用循环不断执行脚本，并1s执行一次

2）修改脚本

在程序运行过程中修改脚本文将ReloadingTest.txt中的输出

```
String sayHello() {
    def greet = "Hello, world123!"
    greet
}
```

注：将输出由Hello, world!改为Hello, world123!，这时程序重新加载会输出Hello, world123!，即修改后的输出内容。

3）执行查看输出

```
Hello, world!
Hello, world!
...
Hello, world123!
Hello, world123!
...
```