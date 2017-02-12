---
title: 字符流操作类
tags: [basic]
---

字符流操作类--实例
java默认使用Unicode存储字符串，在写入字符流时我们都可以指定写入的字符串的编码。

### 1. CharArrayReader类
操作字符类的类就是CharArrayReader,CharArrayWriter类，对应的处理字节型数据的流ByteArrayOutputStream。

字符缓冲区，一般将字符串放入到操作字符的io流一般方法是：

```
CharArrayReader reader=mew CharArrayReader(str.toCharArray()); 
```

### 2. FileReader类
```
public class FileReaderOperator {
    public static void main(String[] args) {
        readFile();
        //readFile2();
        readFile3();
    }
    /**
     * 读取utf-8编码的文件
     */
    public static void readFile(){
        String fileName="E:/testfile/test2.txt";//utf-8编码文件
        FileReader reader=null;
        try{
            reader=new FileReader(fileName);
            char[] buf=new char[1024];
            int len=0;
            while((len=reader.read(buf))!=-1){
                System.out.println(new String(buf,0,len));
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=reader){
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    /**
     * 读取ANSI保存的txt文件
     */
    public static void readFile2(){
        String fileName="E:/testfile/test.txt";//ANSI文件
        BufferedReader br=null;
        try{
            br=new BufferedReader(new FileReader(fileName));
            String s=null;
            while((s=br.readLine())!=null){
                System.out.println(s);//乱码
                System.out.println(new String(s.getBytes(),"GBK"));//乱码，不能正确输入
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=br){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    /**
     * 读取ANSI保存的txt文件
     */
    public static void readFile3(){
        String fileName="E:/testfile/test.txt";//ANSI文件
        BufferedReader br=null;
        try{
            br=new BufferedReader(new InputStreamReader(new FileInputStream(fileName),"GBK"));
            String s=null;
            while((s=br.readLine())!=null){
                System.out.println(s);
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=br){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

注：问题出在FileReader读取文件的过程中，FileReader继承了InputStreamReader，但并没有实现父类中带字符集参数的构造函数，所以FileReader只能按系统默认的字符集来解码，然后在UTF-8 -> GBK ->UTF-8的过程中编码出现损失，造成结果不能还原最初的字符。

### 3. BufferedReader和BufferedWriter
该流最常用的属readLine()方法了，读取一行数据，并返回String。

实例：使用BufferedWriter将多个文件合并成一个文件

```
public class BufferedWriterOperator {
    public static void main(String[] args) {
        contactFiles();
    }
    
    public static void contactFiles(){
        String[] files=new String[]{"E:/testfile/test2.txt","E:/testfile/test3.txt"};
        BufferedWriter writer=null;
        BufferedReader reader=null;
        try{
            writer=new BufferedWriter(new FileWriter("E:/testfile/test5.txt"));
            String str=null;
            for(String fileName:files){
                reader=new BufferedReader(new FileReader(fileName));
                while((str=reader.readLine())!=null){
                    writer.write(str);
                    writer.newLine();
                }
            }
            writer.flush();
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=writer){
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(null!=reader){
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 4. Console类
实例代码：

```
public class ConsoleOperator {
    public static void main(String[] args) {
        //getDataFromConsole();
        getDataFromConsole2();
    }
    /**
     * 需要在cmd中运行java启动JVM
     */
    public static void getDataFromConsole(){
        Console console=System.console();
        if(null!=console){
            String username=console.readLine("输入用户名：");
            String password=new String(console.readPassword("输入密码："));
            console.printf(username+"密码"+password);
        }else{
            System.out.println("Console is unavailable.");
        }
    }
    /**
     * Eclipse控制台
     */
    public static void getDataFromConsole2(){
        BufferedInputStream bis=null;
        try{
            byte[] buf=new byte[1024];
            bis=new BufferedInputStream(System.in);
            System.out.println("请输入用户名：");
            int len=bis.read(buf);
            String username=new String(buf,0,len);
            System.out.println("请输入密码：");
            len=bis.read(buf);
            String password=new String(buf,0,len);
            System.out.println(username+":"+password);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null!=bis){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

注：Java要与Console进行交互，不总是能得到可用的Java Console类的。一个JVM是否有可用的Console，依赖于底层平台和JVM如何被调用。如果JVM是在交互式命令行（比如Windows的cmd）中启动的，并且输入输出没有重定向到另外的地方，那么就我们可以得到一个可用的Console实例。

注2：Console 只能用在标准输入、输出流未被重定向的原始控制台中使用，在 Eclipse 或者其他 IDE 的控制台是用不了的。

### 5. System中的out，in和err
JVM启动的时候通过Java运行时初始化这3个流，所以你不需要初始化它们(尽管你可以在运行时替换掉它们)。

1）System.in
System.in是一个典型的连接控制台程序和键盘输入的InputStream流。通常用于接收数据通过命令行参数传入系统。

2）System.out
System.out是一个PrintStream流。System.out一般会把你写到其中的数据输出到控制台上。System.out通常仅用在类似命令行工具的控制台程序上。System.out也经常用于打印程序的调试信息(尽管它可能并不是获取程序调试信息的最佳方式)。
 
3）System.err
System.err是一个PrintStream流。System.err与System.out的运行方式类似，但它更多的是用于打印错误文本。一些类似Eclipse的程序，为了让错误信息更加显眼，会将错误信息以红色文本的形式通过System.err输出到控制台上。

4）实例

```
try {
    InputStream input = new FileInputStream("c:\\data\\...");
    System.out.println("File opened...");
} catch (IOException e) {
    System.err.println("File opening failed:");
    e.printStackTrace();
}
```

5）替换系统流
使用System.setIn(), System.setOut(), System.setErr()方法设置新的系统流

```
OutputStream output = new FileOutputStream("c:\\data\\system.out.txt");
PrintStream printOut = new PrintStream(output);
System.setOut(printOut);
```

参考：http://blog.csdn.net/yczz/article/details/38761237
