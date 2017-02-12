---
title: 字节流操作类
tags: [basic]
---

字节流操作类--实例

### 1. 文件操作类
实例代码：

```
public class FileOperator {
    public static void main(String[] args) {
        createFile();
        System.out.println(getCreateDate());
    }
    /**
     * 创建文件
     */
    public static void createFile(){
        File file=new File("E:/testfile/test.txt");
        try {
            //系统路径必须存在，方法返回true则表示文件不存在并成功创建，返回false表示文件存在
            boolean flag=file.createNewFile();
            System.out.println(file.length());//文件本身大小
            System.out.println(new Date(file.lastModified()));//最后创建时间
            System.out.println("文件分区大小"+file.getTotalSpace()/(1024)+"K");
            //file.mkdirs();  //创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。会将文件名也作为目录创建
            //file.delete(); //  删除此抽象路径名表示的文件或目录 
            System.out.println("文件名  "+file.getName());  //返回由此抽象路径名表示的文件或目录的名称。  
            System.out.println("文件父目录字符串 "+file.getParent());//返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    /**
     * 获取文件创建日期
     * @return
     */
    public static Date getCreateDate(){
        //dir E:\testfile\test.txt /tc，参数/t表示时间区段，c表示创建的时间
        String filePath = "E:\\testfile\\test.txt";//E:/testfile/test.txt，在windows系统中这种路径是无法正确输出信息的
        String strTime = null;
        try {
            Process p = Runtime.getRuntime().exec("cmd /C dir "+ filePath + "/tc" );
            InputStream is = p.getInputStream();
            BufferedReader br = new BufferedReader(new InputStreamReader(is));
            String line;
            while((line = br.readLine()) != null){
                if(line.endsWith(".txt")){//文件后缀
                    strTime = line.substring(0,17);
                    break; 
                }
             }
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("创建时间    " + strTime);
        if(null!=strTime){
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy/MM/dd  HH:mm");//两个空格
            try {
                return sdf.parse(strTime);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```

注：File类中无法获取文件的创建时间属性，只能通过系统命令返回文件创建时间等信息。

### 2. FileInputStream类
实例代码：

```
/**
 * 文件输入流常用操作
 */
public class FileInputStreamOperator {
    public static void main(String[] args) {
        //System.out.println(fileLength());
        //System.out.println(fileLength2());
        //getFileContent();
        //getFileContent2();
        //getFileContent3();
        markStream();
    }
    /**
     * 获取文件长度
     * @return
     */
    public static int fileLength(){
        String filePath="E:/testfile/test.txt";
        int count=0;
        InputStream is=null;
        try {
            is=new FileInputStream(new File(filePath));
            System.out.println(is.available());//输出文件的字节数
            while(is.read()!=-1){
                count++;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=is){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return count;
    }
    
    /**
     * 获取文件长度,使用is.read()每次都读取一个字节，当文件比较庞大时，处理起来非常浪费资源
     * 引出了缓冲区的概念。可以一次读取的字节数目的长度
     */
    public static int fileLength2(){
        String filePath="E:/testfile/test.txt";
        int count=0;
        InputStream is=null;
        try {
            byte[] buf=new byte[1024];
            is=new FileInputStream(new File(filePath));
            int len=0;
            while((len=is.read(buf))!=-1){
                count+=len;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=is){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return count;
    }
    /**
     * 输出文件内容，中文文件设置读入的字符编码
     * 向test.txt文件中写入中文。
     * 中文乱码--windows系统下保持的文件默认使用的是GBK/GB2312编码，而java处理使用的是UTF-8，因此出现编码错误
     * 修改：文件另存为选择utf-8编码，另外可在字节输入流中设置编码
     * 
     */
    public static void getFileContent(){
        String filePath="E:/testfile/test.txt";
        BufferedReader br=null;
        try {
            br=new BufferedReader(new FileReader(new File(filePath)));
            String s=null;
            while((s=br.readLine())!=null){
                System.out.println(s);
            }
        } catch (Exception e) {
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
     * 字节输入流获取数据，这种方式不适合处理字符型文本
     * 通过：new String方式指定编码格式化字节数组
     */
    public static void getFileContent2(){
        String filePath="E:/testfile/test.txt";
        FileInputStream fis=null;
        try {
            fis=new FileInputStream(new File(filePath));
            byte[] buf=new byte[1024];
            int len=0;
            while((len=fis.read(buf))!=-1){
                System.out.println(new String(buf,0,len,"GBK"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=fis){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    /**
     * 通过InputStreamReader指定编码
     */
    public static void getFileContent3(){
        String filePath="E:/testfile/test.txt";
        BufferedReader br=null;
        try {
            br=new BufferedReader(new InputStreamReader(new FileInputStream(new File(filePath)),"GBK")) ;
            String s=null;
            while((s=br.readLine())!=null){
                System.out.println(s);
            }
        } catch (Exception e) {
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
     * mark方法的使用
     * FileInputStream不支持mark/reset操作；BufferedInputStream支持此操作
     * 
     */
    public static void markStream(){
        String filePath="E:/testfile/test.txt";
        FileInputStream fis=null;
        BufferedInputStream bis=null;
        try {
            fis=new FileInputStream(new File(filePath));
            if(fis.markSupported()){//FileInputStream本身不支持mark
                System.out.println("FileInputStream支持mark");
            }else{
                System.out.println("FileInputStream不支持mark");
            }
            bis=new BufferedInputStream(fis);
            if(bis.markSupported()){
                System.out.println("BufferedInputStream支持mark");
                bis.mark(8);//设置标记
                byte[] buf=new byte[1024];
                int len=0;
                while((len=bis.read(buf))!=-1){
                    System.out.println(new String(buf,0,len,"GBK"));
                    bis.reset();//重定向输入流
                }
            }else{
                System.out.println("BufferedInputStream不支持mark");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=fis){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

注：read方法一次读取一个字节，效率低，引入缓存read(buf)，一次可读取一个缓冲区大小的字节，效率更高。

注2：mark(readlimit)的含义是在当前位置作一个标记，制定可以重新读取的最大字节数，也就是说你如果标记后读取的字节数大于readlimit，你就再也回不到回来的位置了。通常InputStream的read()返回-1后，说明到达文件尾，不能再读取。除非使用了mark/reset。

### 3. FileOutputStream类实现文件复制操作
Java I/O默认是不缓冲流的，所谓"缓冲"就是先把从流中得到的一块字节序列暂存在一个被称为buffer的内部字节数组里，然后你可以一下子取到这一整块的字节数据，没有缓冲的流只能一个字节一个字节读，效率孰高孰低一目了然。

操作代码：

```
public class FileOutputStreamOperator {
    public static void main(String[] args) {
        //fileCopy();
        //fileCopy2();
        fileCopy3();
    }
    /**
     * 复制文件，使用字节数组缓冲区
     */
    public static void fileCopy2(){
        String filePath="E:/testfile/test.txt";
        String copyFile="E:/testfile/testcopy.txt";
        FileInputStream fis=null;
        FileOutputStream fos=null;
        try {
            fis=new FileInputStream(new File(filePath));
            fos=new FileOutputStream(new File(copyFile));
            int len=0;
            byte[] buf=new byte[1024];
            while((len=fis.read(buf))!=-1){
                fos.write(buf,0,len);//写入低字节
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=fis){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(null!=fos){
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    public static void fileCopy(){
        String filePath="E:/testfile/test.txt";
        String copyFile="E:/testfile/testcopy.txt";
        FileInputStream fis=null;
        FileOutputStream fos=null;
        try {
            fis=new FileInputStream(new File(filePath));
            fos=new FileOutputStream(new File(copyFile));
            int b;
            while((b=fis.read())!=-1){
                fos.write(b);//写入低字节
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=fis){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(null!=fos){
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    /**
     * 使用BufferedInputStream和BufferedOutputStream类实现缓存
     */
    public static void fileCopy3(){
        String filePath="E:/testfile/test.txt";
        String copyFile="E:/testfile/testcopy.txt";
        BufferedInputStream bis=null;
        BufferedOutputStream bos=null;
        try {
            bis=new BufferedInputStream(new FileInputStream(new File(filePath)),1024);//指定缓存区大小
            bos=new BufferedOutputStream(new FileOutputStream(new File(copyFile)),1024);
            int b;
            while((b=bis.read())!=-1){
                bos.write(b);//写入低字节
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(null!=bis){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(null!=bos){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        
    }
}
```

注：BufferedOutputStream中read方法的内部实现，实际是从BufferdOutputStream的缓存区中一个一个的读取数据，当缓存区位置大约当前数组能容纳的最大值时，则重新填充缓冲区。

BuffredOutputStream中read方法实现代码：

```
// 读取下一个字节
public synchronized int read() throws IOException {
    // 若已经读完缓冲区中的数据，则调用fill()输入流读取下一部分数据来填充缓冲区
    if (pos >= count) {
        fill();
        if (pos >= count)
            return -1;
    }
    // 从缓冲区中读取指定的字节
    return getBufIfOpen()[pos++] & 0xff;
}
```

### 4. 读写对象：ObjectInputStream 和ObjectOutputStream
该流允许读取或写入用户自定义的类，但是要实现这种功能，被读取和写入的类必须实现Serializable接口，其实该接口并没有什么方法，可能相当于一个标记而已。(代码参考java序列化实现)

### 5. DataInputStream、DataOutputStream来写入或读出数据
有时没有必要存储整个对象的信息，而只是要存储一个对象的成员数据，成员数据的类型假设都是Java的基本数据类型，这样的需求不必使用到与Object输入、输出相关的流对象。DataInputStream的好处在于在从文件读出数据时，不用费心地自行判断读入字符串时或读入int类型时何时将停止，使用对应的readUTF()和readInt()方法就可以正确地读入完整的类型数据。

实例：将Member类实例的成员数据写入文件中，并打算在读入文件数据后，将这些数据还原为Member对象。

```
public class DataInputStreamOperator {
    public static void main(String[] args) {
        putData();
        getData();
    }
    /**
     * 将数据写入到文件中
     */
    public static void putData(){
        Member[] members = {new Member("Justin",90),new Member("momor",95),new Member("Bush",88)};
        String fileName="E:/testfile/datatest.txt";
        DataOutputStream dataOutputStream=null;
        try
        {
            dataOutputStream = new DataOutputStream(new FileOutputStream(fileName));  
            for(Member member:members)  
            {  
                //写入UTF字符串  
               dataOutputStream.writeUTF(member.getName());  
               //写入int数据  
               dataOutputStream.writeInt(member.getAge());  
            }
            //所有数据至目的地  
            dataOutputStream.flush();
        }catch(Exception e){
            e.printStackTrace();
        }
        finally{
            if(null!=dataOutputStream){
                try {
                    dataOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    /**
     * 从文件中获取数据
     */
    public static void getData(){
        Member[] members = new Member[3];
        String fileName="E:/testfile/datatest.txt";
        DataInputStream dataInputStream =null;
        try{
            dataInputStream = new DataInputStream(new FileInputStream(fileName));  
            //读出数据并还原为对象  
            for(int i=0;i<members.length;i++)  
            {  
               //读出UTF字符串  
               String name =dataInputStream.readUTF();  
               //读出int数据  
               int score =dataInputStream.readInt();  
               members[i] = new Member(name,score);  
            }
            //显示还原后的数据  
            for(Member member : members)  
            {  
               System.out.printf("%s\t%d%n",member.getName(),member.getAge());  
            }
        }catch(Exception e){  
            e.printStackTrace();  
        }
        finally{
            //关闭流  
            try {
                dataInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }  
        }
    }
}
class Member {  
    private String name;  
    private int age;  
    public Member() {  
    }  
   public Member(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
    public void setName(String name){  
        this.name = name;  
    }  
    public void setAge(int age) {  
        this.age = age;  
    }  
    public String getName() {  
        return name;  
    }  
    public int getAge() {  
        return age;  
    }  
} 
```

DataOutputStream类提供三个写入字符串的方法：

1）public final void writeBytes(String s)  //由于JAVA的字符编码是Unicode的，第个字符占两个字节，writeBytes方法只是将每个字符的低字节写入到目标设备中.

2）public final void writeChars(String s)，writeChars是将字符的两个字节都写入到目标设备中

3）public final void writeUTF(String str)，writeUTF将字符串按照UTF编码写入到目标设备(其中包括长度)

### 6. PushbackInputStream类
java.io.PushbackInputStream拥有一个PushBack缓冲区，从PushbackInputStream读出数据后，只要PushBack缓冲区没有满，就可以使用unread()将数据推回流的前端。

实例场景

1）假设一个文本文件中同时存储有ASCII码范围的英文字符与BIG5范围的中文字符。想要判断那些位置是ASCII而哪些位置是BIG5中文字符，BIG5中文字符使用两个字节来表示一个中文字，而ASCII只使用一个字节来表示英文字符。

2）Big5中文为了与ASCII兼容，低字节范围内0xA4-0xF9，而高字节为0x40--0x7E以及0xA1--0xFE。存储时低字节先存，再存高字节，所以读取时只要先读到字节是在0xA4--0xF9，就表示它可能是一个中文字符的前半数据。

3）下面的范例说明PushbackInputStream的功能，一次从文件中读取两个字节，并检查两个字节合并后的整数值是否在0xA440--0xFFFF之间，这样可以简单地判断其两个字节合并后是否为BIG码。如果是BIG5码则使用这两个字节产生String实例以显示汉字字符；如果不在这个范围之内，则可能是个ASCII范围内的字符，您可以显示第一个字节的字符表示，并将第二个字节推回流，以待下一次可以重新读取。

实例代码：

```
public class PushbackInputStreamDemo {
    public static void main(String[] args) throws UnsupportedEncodingException {
        putBig5Data();
        getBig5Data2();
        getBig5Data3();
        getBig5Data();
        //System.out.println(Charset.defaultCharset());//JVM系统默认编码
    }
    /**
     * 从文件中读取big5数据
     */
    public static void getBig5Data(){
        String fileName="E:/testfile/big5.txt";
        PushbackInputStream pushbackInputStream=null;
        try  
        {
            pushbackInputStream =   new PushbackInputStream(new FileInputStream(fileName));  
            byte[] array = new byte[2];  
            int tmp = 0;
            while(pushbackInputStream.read(array)!=-1)  
            {  
                //两个字节转换为整数   
                tmp = (short)((array[0] << 8) | (array[1] & 0xff));  
                tmp = tmp & 0xFFFF;
                //判断是否为BIG5，如果是则显示BIG5中文字  
                if(tmp >= 0xA440 && tmp < 0xFFFF)  
                {  
                    System.out.println("BIG5:" + new String(array,"BIG5"));  
                }  
                else  
                {
                    //将第二个字节推回流  
                    pushbackInputStream.unread(array,1,1);
                    //显示ASCII范围的字符  
                    System.out.println("ASCII: " + (char)array[0]);  
                }
            }  
        }  
        catch(Exception e)  
        {
            System.out.println("请指定文件名称");  
        }finally{
             try {
                pushbackInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }  
        } 
    }
    /**
     * 向文件中写big5数据
     * @throws UnsupportedEncodingException 
     */
    public static void putBig5Data() throws UnsupportedEncodingException{
        String fileName="E:/testfile/big5.txt";
        String content="hello祖國";//utf8兼容汉字编码
        //String content="hello祖国";//国不能正确找到big5对应的汉字会出现乱码
        StringReader reader=null;
        BufferedWriter writer=null;
        try{
            reader=new StringReader(content);
            writer=new BufferedWriter(new OutputStreamWriter((new FileOutputStream(fileName)),"BIG5"));
            int r;
            while((r=reader.read())!=-1){
                writer.write(r);
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
        }
    }
    /**
     * 读取文档内容
     */
    public static void getBig5Data2(){
        String fileName="E:/testfile/big5.txt";
        BufferedInputStream bis=null;
        try{
            bis=new BufferedInputStream(new FileInputStream(fileName));
            byte[] buf=new byte[1024];
            int len=0;
            while((len=bis.read(buf))!=-1){
                System.out.println(len);
                String s=new String(buf,0,len,"BIG5");
                System.out.println(s);
                System.out.println(s.getBytes("BIG5").length);
            }
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
    
    /**
     * 读取文档内容
     */
    public static void getBig5Data3(){
        String fileName="E:/testfile/big5.txt";
        BufferedReader reader=null;
        try{
            reader=new BufferedReader(new InputStreamReader((new FileInputStream(fileName)),"BIG5"));
            char[] buf=new char[1024];
            int len=0;
            while((len=reader.read(buf))!=-1){
                System.out.println(len);//7个字符
                String s=new String(buf,0,len);
                System.out.println(s);
                System.out.println(s.getBytes("BIG5").length);//9个字节
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
}
```

### 7. SequenceInputStream类操作
当我们需要从多个输入流中向程序读入数据。此时，可以使用合并流，将多个输入流合并成一个SequenceInputStream流对象。SequenceInputStream会将与之相连接的流集组合成一个输入流并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止。合并流的作用是将多个源合并合一个源。其可接收枚举类所封闭的多个字节流对象。

实例：从多个文件中读取数据

```
public class SequenceInputStreamOperator {
    public static void main(String[] args) {
        readFromFiles();
    }
    
    public static void readFromFiles(){
        String file1="E:/testfile/test.txt";
        String file2="E:/testfile/test2.txt";
        SequenceInputStream sis=null;
        try{
            Vector<InputStream> v=new Vector<InputStream>();
            v.add(new FileInputStream(file1));
            v.add(new FileInputStream(file2));
            sis=new SequenceInputStream(v.elements());
            byte[] buf=new byte[1024];
            int len=0;
            while((len=sis.read(buf))!=-1){
                System.out.println(new String(buf,0,len));
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            try {
                sis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 8. PrintStream类
PrintStream类提供了一系列的print和println方法，可以将基本数据类型的数据格式化成字符串输出。与PrintStream对应的PrintWriter类,即使遇到了文本换行标识符(n),PrintWriter类也不会自动清空缓冲区。PrintWriter的println方法能根据操作系统的不同而生成相应的文本换行标识符。在Windows下的文本换行符是"\r\n"，而Linux下的文本换行标识符是"\n"

注：System.out这个对象就是PrintStream。

### 9. 字节流与字符流的转换
1）InputStreamReader和OutputStreamWriter，是用于将字节流转换成字符流来读写的两个 类。

2）InputStreamReader可以将一个字节流中的字节编码成字符后读取。

3）OutputStreamWriter将字符解码成字节后写入到一个字节流中。

4）避免频繁地在字符与字节间进行转换，最好不要直接使用InputStreamReader和OutputStreamWriter类来读写数据，应尽量使用BufferedWriter类包装OutputStreamWriter类，用BufferedReader类包装 InputStreamReader。

### 10. 总结
1）只要是处理纯文本数据，就优先考虑使用字符流。 除此之外都使用字节流。

2）InputStream 是所有的输入字节流的父类，它是一个抽象类。

3）ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，它们分别从Byte数组、StringBuffer、和本地文件中读取数据。

参考：http://blog.csdn.net/yczz/article/details/38761237
