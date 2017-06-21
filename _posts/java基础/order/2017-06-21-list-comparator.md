---
title: java集合排序，使用Comparator接口
tags: [basic]
---

在处理集合数据时，有时需要将数据先进行排序，然后按顺序执行。

### 测试实例

实现按用户的出生日期进行排序，年龄大的在前。

```
public class Student2 {
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
    @Override
    public String toString() {
        return "Student [name=" + name + ", birthday=" + birthday
                + ", address=" + address + "]";
    }
}
```

测试输出集合的顺序

```
public class OrderTest2 {
    public static void main(String[] args) throws ParseException {
        Set<Student2> set=new HashSet<Student2>();
        for(int i=0;i<10;i++){
            set.add(genStudent(i));
        }
        System.out.println(Arrays.toString(set.toArray()));
    }
    /**
     * 获取对象
     * @param index
     * @return
     * @throws ParseException
     */
    public static Student2 genStudent(int index) throws ParseException{
        DateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Student2 Student2=new Student2();
        Student2.setName("张三");
        Student2.setAddress("北京"+Math.random());
        Student2.setBirthday(getDate(format.parse("1956-12-03"),index));
        return Student2;
    }
    /**
     * 日期加年
     * @param date
     * @param year
     * @return
     */
    public static Date getDate(Date date,int year){
        Calendar c=Calendar.getInstance();
        c.setTime(date);
        c.add(Calendar.YEAR, year);
        return c.getTime();
    }
}
```

### 创建Comparator接口实现类

```
public class StudentComparator implements Comparator<Student2>{

    @Override
    public int compare(Student2 stu1, Student2 stu2) {
        int result=0;
        if(null!=stu1&&null!=stu2){
            if(null!=stu1.getBirthday()){//先按照日期排序
                result=stu1.getBirthday().compareTo(stu2.getBirthday());
            }
            if(result==0&&null!=stu1.getName()){//再按照姓名排序
                result=stu1.getName().compareTo(stu2.getName());
            }
        }
        return result;
    }

}
```

输出测试

```
public class OrderTest2 {
    public static void main(String[] args) throws ParseException {
        Set<Student2> set=new HashSet<Student2>();
        for(int i=0;i<10;i++){
            set.add(genStudent(i));
        }
        Student2[] arr=new Student2[set.size()];
        Comparator<Student2> comparator=new StudentComparator();
        Arrays.sort(set.toArray(arr),comparator);
        System.out.println(Arrays.toString(arr));
        
        List<Student2> list=new ArrayList<Student2>(set.size());
        list.addAll(set);
        set.clear();
        Collections.sort(list,comparator);
        System.out.println(Arrays.toString(list.toArray()));
    }
    /**
     * 获取对象
     * @param index
     * @return
     * @throws ParseException
     */
    public static Student2 genStudent(int index) throws ParseException{
        DateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Student2 Student2=new Student2();
        Student2.setName("张三");
        Student2.setAddress("北京"+Math.random());
        Student2.setBirthday(getDate(format.parse("1956-12-03"),index));
        return Student2;
    }
    /**
     * 日期加年
     * @param date
     * @param year
     * @return
     */
    public static Date getDate(Date date,int year){
        Calendar c=Calendar.getInstance();
        c.setTime(date);
        c.add(Calendar.YEAR, year);
        return c.getTime();
    }
}
```