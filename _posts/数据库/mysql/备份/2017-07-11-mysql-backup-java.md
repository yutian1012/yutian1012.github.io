---
title: mysql备份--java实现
tags: [database]
---

Procss类的输入和输出流的理解，要学会换位思考。从JAVA程序角度理解输入和输出流。Process类的getInputStream理解为进程将内容输入到Java程序中，Process类的getOutputStream理解为Java程序将内容输出给Process进程。

### 原理分析

1）数据库备份命令

```
D:/Program Files/mysql-5.6.23-winx64/bin/mysqldump -h 192.168.3.53 -uroot -proot eip-saas --set-gtid-purged=OFF
```

首先通过一个进程Process执行数据库备份命令，然后获取该进程的输出，最后将输出内容输入到一个文件中。

Process类的getInputStream()为了用户与进程交换提供一个输入流。这里的输入流不能理解为进程执行命令的输入流，而要理解为用户为了获取Process输出内容而使用的输入流（即将Process的输出读入到java程序中）视角要放到java程序端。

例如，Process将输出内容先输出到内存的某个地方，这个地方java程序是不知道的，为了让java程序能够访问这部分内容，提供了java程序的输入流来读取这部分内存空间。这样java程序中就能进一步处理Process的输出了。

2）数据库还原思路

```
D:/Program Files/mysql-5.6.23-winx64/bin/mysql.exe -h 192.168.3.53 --default-character-set=utf8 -uroot -proot eip-saas

...

insert into xxx(xxx) values (xxx);//执行导入sql操作
```

首先通过一个进程Process来执行mysql的连接（获取mysql的客户端），然后在客户端上执行sql命令。

Process类的getOutputStream()是为了用户与进程交换提供的一个命令接收流。这里的输出流不要理解为进程执行信息输出到控制台上的输出，而要理解为用户将字符串命令输出到进程中（从java程序角度考虑）。进程Process对接此输出流，将其转化为进程Process的输入流，来并执行接收命令。

例如，getOutputStream用户将命令输出到Process能访问的一个内存空间中，那么Process进程就能从该内存空间中获取用户的输入命令，进而执行相应的操作。

### 数据库备份工具类

首先从属性文件中加载数据库的连接信息，然后获取mysqldump.exe的执行命令路径，利用该命令执行备份操作。最后，利用进程Process类来连接数据库，执行数据库命令，达到还原数据库的功能。

```
package com.hotent.ipph.util;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.util.Properties;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class DataBaseBakUtil {

    private static String mysqlDumpPath=""; //mysql下的bin路径
    private static String mysqlDumpCmd="";  //mysql备份数据命令
    private static String mysqlRestoreCmd="";  //mysql还原数据命令
    private static String url="";  //系统连接数据库地址，里面包含ip地址，端口号，数据库名
    private static String host=""; //ip地址
    private static String port=""; //端口号
    private static String user=""; //数据库用户名
    private static String password=""; //数据库密码
    private static String database=""; //数据库名
    private static String gt5_6="";  //判断是不是mysql 5.6版本
    private static boolean mysqlVersionGT5_6=true;
    /**
     * 读取配置文件信息，获取相应的参数值
     */
    public static void init(){
        try {
            //获取属性配置文件，如果使用spring，也可以直接访问相应的bean
            InputStream in = DataBaseBakUtil.class.getClassLoader().getResourceAsStream("conf/app.properties");
            Properties p = new Properties();
            p.load(in);
            mysqlDumpPath=p.getProperty("db.mysqldump.path");
            mysqlDumpCmd=p.getProperty("db.mysqldump.cmd");
            mysqlRestoreCmd=p.getProperty("db.mysqlrestore.cmd");
            gt5_6=p.getProperty("mysql.version.gt5_6");
            url = p.getProperty("jdbc.url");  
            user = p.getProperty("jdbc.username"); 
            password = p.getProperty("jdbc.password"); 
            //通过url获取对应的ip地址，端口号，及数据库名
            host=getMysqlHostFromUrl(url);
            //若连接的数据库在本地，且以localhost连接，则获取不到，需加判断设值。
            if(host==null || "".equals(host)){
                host="localhost";
            }
            port=getMysqlPort(url);
            database=getMysqlDatabase(url);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    /**
     * 备份mysql数据库
     * @param file
     * @throws IOException
     */
    public static void backup(String file)throws Exception {
        init();
        InputStream in=null;
        InputStreamReader inputStreamReader=null;
        BufferedReader bufferedReader =null;
        FileOutputStream fout =null;
        OutputStreamWriter writer=null;
        try {
            Runtime runtime = Runtime.getRuntime();
            // 调用 调用mysql的安装目录的命令C:\Program Files\MySQL\MySQL Server 5.6\bin,可在dos中先运行测试
            if(null==host||"".equals(host)){
                //解析url获取值
                host=getMysqlHostFromUrl(url);
            }
            if(null==database||"".equals(database)){
                database=getMysqlDatabase(url);
            }
            if(null==port||"".equals(port)){
                port=getMysqlPort(url);
            }
            
            if(null!=mysqlDumpPath&&null!=mysqlDumpCmd&&null!=host&&null!=user&&null!=password&&null!=database){
                if(isCmdExists(mysqlDumpPath)){
                    StringBuffer cmd=new StringBuffer();
                    cmd.append(mysqlDumpPath).append(mysqlDumpCmd).append(" -h "+host).append(" -u"+user);
                    if(null!=port&&!"3306".equals(port)) cmd.append(" -P"+port);
                    cmd.append(" -p"+password).append(" "+database);
                    if(null==gt5_6&&mysqlVersionGT5_6||"true".equals(gt5_6)){
                        cmd.append(" --set-gtid-purged=OFF");
                    }
                    //Process p = runtime.exec("D:/Program Files/mysql-5.6.23-winx64/bin/mysqldump -h 192.168.3.53 -uroot -proot eip-saas --set-gtid-purged=OFF");
                    Process p=runtime.exec(cmd.toString());
                    //执行备份时的错误提示：Couldn't execute 'SELECT @@GTID_MODE': Unknown system variable 'GTID_MODE' (1193)GTIDs可以让主从结构复制的跟踪和比较变得简单。,--set-gtid-purged=OFF仅限在mysql 5.6中使用
                    // 设置导出编码为utf-8。这里必须是utf-8,把进程执行中的控制台输出信息写入.sql文件，即生成了备份文件。注：如果不对控制台信息进行读出，则会导致进程堵塞无法运行
                    in= p.getInputStream();// 控制台的输出信息作为输入流
                    
                    inputStreamReader= new InputStreamReader(in, "UTF-8");//设置编码为utf-8
                    // 设置输出流编码为utf-8。这里必须是utf-8，否则从流中读入的是乱码
                    String sqlString;
                    StringBuffer sb = new StringBuffer("");
                    String outString;
                    // 组合控制台输出信息字符串
                    bufferedReader= new BufferedReader(inputStreamReader);
                    while ((sqlString = bufferedReader.readLine()) != null) {
                        sb.append(sqlString + "\r\n");
                    }
                    outString = sb.toString();
                    
                    // 要用来做导入用的sql目标文件：
                    fout= new FileOutputStream(file);//输出sql的文件
                    writer = new OutputStreamWriter(fout, "UTF-8");
                    writer.write(outString);
                    writer.flush();
                }else{
                    throw new FileNotFoundException("未找到mysqldump执行命令");
                }
            }
        }
        finally{
            if(null!=bufferedReader) bufferedReader.close();
            if(null!=inputStreamReader) inputStreamReader.close();
            if(null!=in) in.close();
            if(null!=writer) writer.close();
            if(null!=fout) fout.close();
        }
    }
    /**
     * 还原mysql数据库
     * @throws IOException 
     */
    public static void load(String file) throws Exception {//传递待还原的sql文件
        init();
        OutputStream out=null;
        BufferedReader br=null;
        OutputStreamWriter writer=null;
        try {
            if(null==host&&null==database&&null!=url){
                //解析url获取值
                host=getMysqlHostFromUrl(url);
                database=getMysqlDatabase(url);
            }
            if(null!=mysqlDumpPath&&null!=mysqlRestoreCmd&&null!=host&&null!=user&&null!=password&&null!=database){
                if(isCmdExists(mysqlDumpPath+File.separator+mysqlRestoreCmd)){
                    Runtime runtime = Runtime.getRuntime();
                    StringBuffer cmd=new StringBuffer();
                    //cmd.append(mysqlDumpPath).append("/").append(mysqlRestoreCmd).append(" -h "+host).append(" -u"+user).append(" -p"+password).append(" "+database);
                    cmd.append(mysqlDumpPath).append(mysqlRestoreCmd).append(" -h "+host).append(" --default-character-set=utf8").append(" -u"+user).append(" -p"+password).append(" "+database);
                    //Process p = runtime.exec("D:/Program Files/mysql-5.6.23-winx64/bin/mysql.exe -h 192.168.3.53 --default-character-set=utf8 -uroot -proot eip-saas");
                    Process p=runtime.exec(cmd.toString());
                    out= p.getOutputStream();//控制台的输入信息作为输出流 
                    String inputString;
                    StringBuffer sb = new StringBuffer("");
                    String outString;
                    br = new BufferedReader(new InputStreamReader(new FileInputStream(file), "utf-8"));//读取还原的sql文件
                    while ((inputString = br.readLine()) != null) {
                        sb.append(inputString + "\r\n");
                    }
                    outString = sb.toString();
                    writer = new OutputStreamWriter(out, "utf-8");
                    writer.write(outString);
                    // 注：这里如果用缓冲方式写入文件的话，会导致中文乱码，用flush()方法则可以避免      
                    writer.flush();
                }
            }
        } catch (Exception e) {
            //e.printStackTrace();
            throw new Exception("数据库还原失败"); 
        }
        finally{
            if(null!=writer) writer.close();
            if(null!=out) out.close();
            if(null!=br) br.close();
        }
    }
    
    private static String getMysqlHostFromUrl(String url){
        if(null!=url&&!"".equals(url)){
            Pattern p=Pattern.compile(".*://(\\d{0,3}\\.\\d{0,3}\\.\\d{0,3}\\.\\d{0,3}):.*");
            Matcher m=p.matcher(url);
            if(m.matches()){
                if(m.groupCount()>0){
                    return m.group(1);
                }
            }
        }
        return null;
    }
    
    private static String getMysqlDatabase(String url){
        if(null!=url&&!"".equals(url)){
            if(url.indexOf("/")!=-1){
                return url.substring(url.lastIndexOf("/")+1);
            }
        }
        return null;
    }
    
    private static String getMysqlPort(String url){
        if(null!=url&&!"".equals(url)){
            if(url.indexOf(":")!=-1){
                return url.substring(url.lastIndexOf(":")+1,url.lastIndexOf("/"));
            }
        }
        return null;
    }
    
    private static boolean isCmdExists(String path){
        File cmd=new File(path);
        return cmd.exists();
    }
}

```


