---
title: JAXB日期和数字格式化输出
tags: [xml]
---

参考：http://www.cnblogs.com/yjmyzz/p/xstream-jaxb-format-date-and-number.html

Jaxb是java中用于对象xml序列化、反序列化的经典开源项目，利用它们将对象转换成xml时，经常会遇到日期(Date)、数字按指定格式输出的需求。

注：jaxb处理要先创建一个Adapter，然后要处理类的属性成员Date字段的get方法上使用创建的Adapter。

### 日期的格式化操作

1）新建Adapter类

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.xml.bind.annotation.adapters.XmlAdapter;

public class JaxbDateAdapter extends XmlAdapter<String, Date> {
    static final String STANDARM_DATE_FORMAT = "yyyy-MM-dd";
    private DateFormat format = new SimpleDateFormat(STANDARM_DATE_FORMAT);
    
    @Override
    public Date unmarshal(String v) throws Exception {
        if (v == null) {
            return null;
        }
        return format.parse(v);
    }

    @Override
    public String marshal(Date v) throws Exception {
        return format.format(v);
    }
}
```

2）将Adapter应用于字段上

```
import java.util.Date;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;
import javax.xml.bind.annotation.adapters.XmlJavaTypeAdapter;
@XmlRootElement
public class Student {
    private String name;
    private int age;
    private Date birthday;
    public Student() {}
    public Student(String name,int age,Date birthday) {
        this.name=name;
        this.age=age;
        this.birthday=birthday;
    }
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
    public Date getBirthday() {
        return birthday;
    }
    @XmlAttribute
    @XmlJavaTypeAdapter(JaxbDateAdapter.class)
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}
```

注：在setBirthday方法上设置注解

3）测试

```
import java.util.Date;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Marshaller;

import org.junit.Test;

/**
 * Unit test for simple App.
 */
public class AppTest{
    @Test
    public void testJaxb() throws JAXBException {
        Student s=new Student("zhangsan",23,new Date());
         JAXBContext context = JAXBContext.newInstance(Student.class);
         Marshaller marshaller=context.createMarshaller();
        // 设置编码字符集  
         marshaller.setProperty(Marshaller.JAXB_ENCODING, "UTF-8");
        // 格式化XML输出，有分行和缩进  
         marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true); 
         marshaller.marshal(s, System.out);
    }
}
```