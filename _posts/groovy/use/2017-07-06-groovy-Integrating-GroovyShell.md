---
title: 将groovy整合到应用中--GroovyShell
tags: [groovy]
---

参考：http://www.groovy-lang.org/integrating.html

### 使用GroovyShell整合

1）运算

方式一：使用GroovyShell对象的evaluate方法计算

```
@Test
void testCaculate(){
    def shell = new GroovyShell()
    def result = shell.evaluate '3*5'
    assert result== 15

    result=shell.evaluate('3+5');
    assert result== 8;

    result=shell.evaluate('3-5');
    assert result==-2;

    result=shell.evaluate('3/5');
    assert result==0.6;
}

```

方式二：使用Script对象的run方法执行脚本

```
@Test
void testCaculate2(){
    def shell = new GroovyShell()
    def script = shell.parse '3*5'
    assert script instanceof groovy.lang.Script
    assert script.run() == 15
    
    script=shell.parse("3+5");
    assert script.run() == 8
    
    script=shell.parse("3-5");
    assert script.run() == -2
    
    script=shell.parse("3/5");
    assert script.run() == 0.6
}
```

2）应用于groovy脚本间共享数据

1、从应用中设置变量到groovy脚本中

使用Binding对象来存储在应用中使用的变量，将binding对象设置到GroovyShell对象中，然后执行对象的脚本，就能取出设置在binding中的值。

```
@Test
void testShareData(){
    def sharedData = new Binding()
    def shell = new GroovyShell(sharedData)
    def now = new Date()
    sharedData.setProperty('text', 'I am shared data!')
    sharedData.setProperty('date', now)
    
    String result = shell.evaluate('"At $date, $text"')
    
    assert result == "At $now, I am shared data!"
}
```

注：脚本中使用变量使用$text，即$符号来引用变量。

2、从Groovy脚本中获取变量

```
@Test
void testShareData2(){
    def sharedData = new Binding()                          
    def shell = new GroovyShell(sharedData)                 

    shell.evaluate('foo=123')                               

    int foo=sharedData.getProperty('foo')//返回object
    
    assert foo == 123
}
```

注：在groovy脚本中定义了一个未声明的foo变量，通过getProperty获取Object对象，可以转成应用的int对象。

3、groovy运行不改变应用对象的值，除非再次从应用对象中获取返回值

```
@Test
void testShareData3(){
    def sharedData = new Binding()
    def shell = new GroovyShell(sharedData)
    
    int i=100;
    
    sharedData.setProperty('i', i)
    
    shell.evaluate('i=i+1')
    
    assert sharedData.getProperty("i")==101

    assert i == 100
    
}
```

4、脚本中的局部变量不能通过Binding对象的getProperty方法获取

```
@Test
void testShareData4(){
    def sharedData = new Binding()
    def shell = new GroovyShell(sharedData)
    
    shell.evaluate('int foo=123')
    
    try {
        sharedData.getProperty('foo')
    } catch (MissingPropertyException e) {
        println "foo is defined as a local variable"
    }
}
```

注1：局部变量在脚本中声明的方式是通过变量名前加def或显示的类型来定义的。

注2：在调用getProperty时不能获取脚本内部定义的局部变量

3）多线程问题

在多线程间共享变量要注意，Binding对象共享的变量在GroovyShell中不是线程安全的。

1、通过定义多个Binding对象，向同一脚本中传递运行，存在线程安全问题

```
@Test
void testShareData5(){
    def shell = new GroovyShell()
    
    def b1 = new Binding(x:3)//这种方式也能实现数据绑定，类似属性设置
    def b2 = new Binding(x:4)
    def script = shell.parse('x = 2*x')
    script.binding = b1
    script.run()
    script.binding = b2
    script.run()
    assert b1.getProperty('x') == 6
    assert b2.getProperty('x') == 8
    assert b1 != b2
}
```

注：这种在同一个线程中执行没有问题，但出现在多线程中时就会有问题了。

```
@Test
void testShareData5(){
    def shell = new GroovyShell()
    
    def b1 = new Binding(x:3)
    def b2 = new Binding(x:4)
    def script = shell.parse('x = 2*x')
    script.binding = b1
    //script.run()
    def t1 = Thread.start { script.run() }//启动线程
    script.binding = b2
    def t2 = Thread.start { script.run() }//启动第二个线程
    //script.run()
    [t1,t2]*.join()//等待两个线程执行结束
    assert b1.getProperty('x') == 6
    assert b2.getProperty('x') == 8
    assert b1 != b2
}
```

解决方法，脚本也要有多个对应

```
@Test
void testShareData6(){
    def shell = new GroovyShell()
    
    def b1 = new Binding(x:3)
    def b2 = new Binding(x:4)

    def script1 = shell.parse('x = 2*x')
    def script2 = shell.parse('x = 2*x')

    assert script1 != script2
    script1.binding = b1
    script2.binding = b2

    def t1 = Thread.start { script1.run() }
    def t2 = Thread.start { script2.run() }

    [t1,t2]*.join()

    assert b1.getProperty('x') == 6
    assert b2.getProperty('x') == 8
    assert b1 != b2
}
```

4）自定义Script

Script可以通过shell的parse获取，也可以用户自定义实现

新建一个Script脚本MyScript

```
abstract class MyScript extends Script{
    private String name
    
    String greet() {
        "Hello, $name!"
    }
}
```

执行自定义脚本greet方法，并设置变量name

```
@Test
void testScript(){
    def config = new CompilerConfiguration()
    //groovyJava.integreate.MyScript，使用类的全限定名
    config.scriptBaseClass = MyScript.class.getName()
    
    def shell = new GroovyShell(this.class.classLoader, new Binding(), config)
    def script = shell.parse('greet()')//解析待执行的脚本方法
    assert script instanceof MyScript
    script.setName('Michel')
    assert script.run() == 'Hello, Michel!'
}
```

注：在使用时需要配置一个编译配置对象，然后设置自定义脚本为scriptBaseClass。创建GroovyShell对象使用该配置对象，并构造出Script对象。