---
title: Java对象和XML转换实例
tags: [xml]
---

以部门员工为例

1）部门类Department

```
import java.util.List;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement(name="department")  
public class Department {  
  
    private String name;    //部门名称  
    private List<Staff> staffs;           // 其实staff是单复同型，这里是加's'是为了区别staff  
      
    public String getName() {  
        return name;  
    }  
    @XmlAttribute  
    public void setName(String name) {  
        this.name = name;  
    }  
    public List<Staff> getStaffs() {  
        return staffs;  
    }  
    @XmlElement(name="staff")  
    public void setStaffs(List<Staff> staffs) {  
        this.staffs = staffs;  
    }  
}  
```

注：注解信息需要设置到属性设置方法上（即set方法）

2）员工类

```
import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;
import javax.xml.bind.annotation.adapters.XmlJavaTypeAdapter;

@XmlRootElement(name="staff")  
public class Staff {
  
    private String name;    // 职员名称  
    private int age;        // 职员年龄  
    private boolean smoker; // 是否为烟民  
    
    private Date birthDay;//出生日期
      
    public String getName() {  
        return name;  
    }  
    @XmlElement  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getAge() {  
        return age;  
    }  
    @XmlElement
    public void setAge(int age) {  
        this.age = age;  
    }  
    public boolean isSmoker() {  
        return smoker;  
    }  
    @XmlAttribute  
    public void setSmoker(boolean smoker) {  
        this.smoker = smoker;  
    }
    @XmlJavaTypeAdapter(JaxbDateAdapter.class)
    public Date getBirthDay() {
        return birthDay;
    }
    @XmlElement
    public void setBirthDay(Date birthDay) {
        this.birthDay = birthDay;
    }  
}
```

3）转换测试

主要实现从java对象转换成xml，并从xml转换成java对象

```
import java.io.ByteArrayOutputStream;
import java.io.StringReader;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.Marshaller;
import javax.xml.bind.Unmarshaller;

public class Main {  
      
    public static void main(String[] args) throws Exception {  
           
        System.out.println(beantoXml());
        
        xmlToBean();
    }
    
    public static void xmlToBean()throws Exception{
        String xml=beantoXml();
        JAXBContext context = JAXBContext.newInstance(Department.class,Staff.class);// 获取上下文对象  
        
        Unmarshaller unmarshaller=context.createUnmarshaller();//根据上下文获取Unmarshaller对象
          
        Department department=(Department)unmarshaller.unmarshal(new StringReader(xml));
        
        System.out.println(department);
    }
    
    /**
     * 将对象转换成xml文件
     * @throws Exception
     */
    public static String beantoXml()throws Exception{
        JAXBContext context = JAXBContext.newInstance(Department.class,Staff.class);// 获取上下文对象  
        // 根据上下文获取marshaller对象
        Marshaller marshaller = context.createMarshaller();   
        // 设置编码字符集  
        marshaller.setProperty(Marshaller.JAXB_ENCODING, "UTF-8");  
        // 格式化XML输出，有分行和缩进
        marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);   
          
        //marshaller.marshal(getSimpleDepartment(),System.out);// 打印到控制台  
          
        ByteArrayOutputStream baos = new ByteArrayOutputStream();  
        marshaller.marshal(getSimpleDepartment(), baos);  
        return new String(baos.toByteArray()); // 生成XML字符串  
    }
      
    /** 
     * 生成一个简单的Department对象 
     * @return 
     * @throws ParseException 
     */  
    private static Department getSimpleDepartment() throws ParseException {  
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
        List<Staff> staffs = new ArrayList<Staff>();  
        Staff stf = new Staff();  
        stf.setName("周杰伦");  
        stf.setAge(30);  
        stf.setSmoker(false); 
        stf.setBirthDay(sdf.parse("1963-12-16"));
        staffs.add(stf);
        stf = new Staff();
        stf.setName("周笔畅");  
        stf.setAge(28);  
        stf.setSmoker(false);  
        stf.setBirthDay(sdf.parse("1962-11-13"));
        staffs.add(stf);
        stf = new Staff();
        stf.setName("周星驰");  
        stf.setAge(40);  
        stf.setSmoker(true);  
        stf.setBirthDay(sdf.parse("1966-10-15"));
        staffs.add(stf);  
          
        Department dept = new Department();  
        dept.setName("娱乐圈");  
        dept.setStaffs(staffs);  
          
        return dept;  
    }  
} 

```
