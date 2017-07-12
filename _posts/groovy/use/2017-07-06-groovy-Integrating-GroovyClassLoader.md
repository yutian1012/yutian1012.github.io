---
title: 将groovy整合到应用中--GroovyClassLoader
tags: [groovy]
---

参考：http://www.groovy-lang.org/integrating.html

### 加载类方式执行

1）使用GroovyClassLoader加载类

GroovyClassLoader是编译和运行时加载class的核心。需要自动加载类并编译，然后运行脚本。

```
@Test
void testGroovyClassLoader(){
    def gcl = new GroovyClassLoader()
    def clazz = gcl.parseClass('class Foo { void doIt() { println "ok" } }')
    assert clazz.name == 'Foo'
    def o = clazz.newInstance()
    o.doIt()
}
```

注：parseClass方法传递了创建class的字符串，解析成Class后，使用newInstance构造出对象，调用对象方法。

2）GroovyClassLoader类容易造成内存泄漏

由GroovyClassLoader加载和创建的对象会一直保持着对实例的引用，因此GC不能够正常回收这些对象。

```
@Test
void testGroovyClassloader2(){
    def gcl = new GroovyClassLoader()
    def clazz1 = gcl.parseClass('class Foo { }')
    def clazz2 = gcl.parseClass('class Foo { }')
    assert clazz1.name == 'Foo'
    assert clazz2.name == 'Foo'
    assert clazz1 != clazz2
}
```

注：由于解析式传递的是string，会重新创建一个新的对象，不会引用已经创建的。即无法跟踪String定义出的class。

改进方法，可将定义类的信息存储到一个文件中，每次从文件中加载获取类信息。

创建一个classtoloadfile.txt文档，文档内容为：

```
class Foo { }
```

运行测试方法

```
@Test
void testGroovyClassloader3(){
    File file=new File("src/groovyJava/integreate/classtoloadfile.txt");
    if(file.exists()){
        def gcl = new GroovyClassLoader()
        def clazz1 = gcl.parseClass(file)                                           
        def clazz2 = gcl.parseClass(new File(file.absolutePath))
        assert clazz1.name == 'Foo'
        assert clazz2.name == 'Foo'
        assert clazz1 == clazz2 
    }
}
```