---
title: java集合中取出重复对象数据
tags: [basic]
---

使用Set集合作为去除重复对象的过渡集合。

### 测试实例

1）新建实体对象Student

```
public class Student {
    private String name;
    private Date birthday;
    private String address;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Date getBirthday() {
        return birthday;
    }
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
    public String getAddress() {
        return address;
    }
    public void setAddress(String address) {
        this.address = address;
    }
}
```

注：这里先不重写equals和hashcode方法

2）测试类

```
public class SetTest {
    public static void main(String[] args) throws ParseException {
        Set<Student> set=new HashSet<Student>();
        for(int i=0;i<10;i++){
            set.add(genStudent());
        }
        System.out.println(set.size());
    }
    
    public static Student genStudent() throws ParseException{
        DateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Student student=new Student();
        student.setName("张三");
        student.setAddress("北京");
        student.setBirthday(format.parse("1956-12-03"));
        return student;
    }
}
```

注：结果输出set集合的大小为10，即重复对象没有去除

### 重写Student对象的equals和hashcode方法

对象使用Student对象的name和birthday两个属性变量值确定是否重复

```
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result
            + ((birthday == null) ? 0 : birthday.hashCode());
    result = prime * result + ((name == null) ? 0 : name.hashCode());
    return result;
}
@Override
public boolean equals(Object obj) {
    if (this == obj)
        return true;
    if (obj == null)
        return false;
    if (getClass() != obj.getClass())
        return false;
    Student other = (Student) obj;
    if (birthday == null) {
        if (other.birthday != null)
            return false;
    } else if (!birthday.equals(other.birthday))
        return false;
    if (name == null) {
        if (other.name != null)
            return false;
    } else if (!name.equals(other.name))
        return false;
    return true;
}
```

注：这里的hashCode和equals方法仅仅对student的name和birthday属性进行了比较和设置，因此即使address不同，也没有关系。

```
public class SetTest {
    public static void main(String[] args) throws ParseException {
        Set<Student> set=new HashSet<Student>();
        for(int i=0;i<10;i++){
            set.add(genStudent());
        }
        System.out.println(set.size());
    }
    
    public static Student genStudent() throws ParseException{
        DateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Student student=new Student();
        student.setName("张三");
        student.setAddress("北京"+Math.random());
        System.out.println(student.getAddress());
        student.setBirthday(format.parse("1956-12-03"));
        return student;
    }
}
```

运行SetTest类，结果输出1
