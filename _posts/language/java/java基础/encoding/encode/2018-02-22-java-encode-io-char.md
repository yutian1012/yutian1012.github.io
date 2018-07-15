---
title: 在java中字符读取文件时的编码
tags: [coding]
---

参考：http://blog.csdn.net/fgstudent/article/details/43191247

首先查看java运行环境中使用的字符编码集。

```
//输出UTF-8
System.out.println(Charset.defaultCharset());
```

注：在IDE环境中可以设置Text file encoding（项目右击--properties）

*java默认字符集的查找顺序*

在new String创建字符串或调用String类的getBytes方法时，默认传递的字符编码查找顺序为，IDE环境中设置的编码，操作系统默认使用的编码。

### 字符输入流

字符流可以看做是一种包装流，它的底层还是采用字节流来读取字节，然后它使用指定的编码方式将读取字节解码为字符。

InputStreamReader在底层还是采用字节流来读取字节，读取字节后它需要一个编码格式来解码读取的字节，如果我们在构造InputStreamReader没有传入编码方式，那么会采用操作系统默认的GBK来解码读取的字节。

还用上面demo.txt的例子，假设demo.txt编码方式为GBK，我们使用如下代码来读取文件：

```
InputStreamReader  in = new InputStreamReader(new FileInputStream("demo.txt"),"GBK");
```

如果把demo.txt编码方式换成UTF-8，那么我们采用这种方式读取时可设置编码为UTF-8，或不设置字符集（IDE中设置了字符集为UTF-8）

```
InputStreamReader  in = new InputStreamReader(new FileInputStream("demo.txt"),"UTF-8");
```

注：InputStreamReader指定解码编码，二者统一就不会出现乱码了。

### 字符输出流。

字符输出流底层还是采用字节输出流来写文件。只是字符输出流根据指定的编码将字符转换为字节数组，然后输出。

```
OutputStreamWriter out = new OutputStreamWriter(new FileOutputStream("dd.txt"),"UTF-8");
```

注：写入的字符将被编码为UTF-8的字节,生成的文件也将是UTF-8格式的文件。