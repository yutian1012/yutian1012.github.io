---
title: jxls帮助文档
tags: [jxls]
---

在sourceforge下载的jxls压缩包中包含doc文件夹，该文件夹中包含对jxls的使用说明。但是提供的html中引用了apis.google.com服务器上的js文件，导致打开缓慢

解决思路，利用文件内容替换方法，将相关的js去掉。

```
package org.jxls.demo;

import java.io.BufferedOutputStream;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStreamReader;

public class RemoveFielContent {
    
    private static String removeText="<script type=\"text/javascript\" src=\"https://apis.google.com/js/plusone.js\"></script>";
    
    public static void main(String[] args) throws Exception {
        String filePath="D:\\work\\software\\jxls-2.4.1\\doc";
        
        File file=new File(filePath);
        
        processFile(file);
    }
    
    public static void processFile(File file) throws Exception {
        if(null==file) {
            return;
        }
        
        if(file.isFile()) {
            
            System.out.println("processing "+ file.getAbsolutePath());
            
            if(file.getName().endsWith(".html")){
                remveContent(file);
            }
            return;
        }
        
        if(file.isDirectory()) {
            File[] files=file.listFiles();
            
            for(File f:files) {
                processFile(f);
            }
        }
    }
    
    public static void remveContent(File file) throws Exception {
        
        FileInputStream fis=new FileInputStream(file);
        
        StringBuilder result=new StringBuilder();
        try(BufferedReader reader=new BufferedReader(new InputStreamReader(fis))){
        
            String temp=null;
            
            while((temp=reader.readLine())!=null) {
                if(temp.indexOf(removeText)==-1) {
                    result.append(temp);
                }
            }
        
        }
        
        //输出到源文件中
        try(BufferedOutputStream bos=new BufferedOutputStream(new FileOutputStream(file))){
            
            bos.write(result.toString().getBytes());
        }
    }
    
}
```