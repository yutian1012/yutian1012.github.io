---
title: maven常见错误
tags: [maven,tools]
---

### 1. 编译时字符编码文件

运行mvn compile命令时，错误信息：编码GBK的不可映射字符

解决方法：在compiler插件中指定utf-8编码

```
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.7</source>
            <target>1.7</target>
            <encoding>UTF8</encoding>
        </configuration>
    </plugin>
</plugins>
```

注：指定encoding字节集。

### 2. 错误: 程序包com.sun.istack.internal不存在/找不到符号

运行mvn compile命令时，错误信息: 程序包com.sun.istack.internal不存在/找不到符号

查看出错的详细信息：

```
mvn compile -X
```

注：在末尾添加-X选项能够实现打印debug输出，便于定位错误信息。如果dos中的输出内容太多，可以选择重定向到文件

```
mvn compile -X > d:\out.txt
```

注：在系统中不小心引入了com.sun包下的类，而该类一般是不会提供给应用程序使用的，是sun公司的java底层类。定位引入该代码位置，发现错误的应用类。

