---
title: java集合排序，使用Comparable接口
tags: [basic]
---

在处理集合数据时，有时需要将数据先进行排序，然后按顺序执行。注意重新compareTo方法时使用谁与谁比较，这会影响到最终的排序结果。



### 测试实例

实现按用户的出生日期进行排序，年龄小的在前。

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
    @Override
    public String toString() {
        return "Student [name=" + name + ", birthday=" + birthday
                + ", address=" + address + "]";
    }
}
```

测试输出集合的顺序

```
public class OrderTest {
    public static void main(String[] args) throws ParseException {
        Set<Student> set=new HashSet<Student>();
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
    public static Student genStudent(int index) throws ParseException{
        DateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Student student=new Student();
        student.setName("张三");
        student.setAddress("北京"+Math.random());
        student.setBirthday(getDate(format.parse("1956-12-03"),index));
        return student;
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

注：set中的对象是无序的。

### Student对象上实现Comparable接口

```
public class Student implements Comparable<Student>{
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
    @Override
    //this对象与student比较时，先后顺序与最终的排序有很大关系
    public int compareTo(Student student) {
        int result=0;
        if(null!=student){
            if(null!=student.getBirthday()){//先按照日期排序
                result=student.getBirthday().compareTo(this.getBirthday());
            }
            if(result==0&&null!=student.getName()){//再按照姓名排序
                result=student.getName().compareTo(this.getName());
            }
        }
        return result;
    }
}
```

注：重写compareTo方法，实现排序的比较

补充

```
//如果是按时间先小后大，年龄小的在前
student.getBirthday().compareTo(this.getBirthday());
//则是按时间的先大后小，年龄大的在前
this.getBirthday().compareTo(student.getBirthday());
```

```
public class OrderTest {
    public static void main(String[] args) throws ParseException {
        Set<Student> set=new HashSet<Student>();
        for(int i=0;i<10;i++){
            set.add(genStudent(i));
        }
        Student[] arr=new Student[set.size()];
        Arrays.sort(set.toArray(arr));
        System.out.println(Arrays.toString(arr));
        
        List<Student> list=new ArrayList<Student>(set.size());
        list.addAll(set);
        set.clear();
        Collections.sort(list);
        System.out.println(Arrays.toString(list.toArray()));
    }
    /**
     * 获取对象
     * @param index
     * @return
     * @throws ParseException
     */
    public static Student genStudent(int index) throws ParseException{
        DateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Student student=new Student();
        student.setName("张三");
        student.setAddress("北京"+Math.random());
        student.setBirthday(getDate(format.parse("1956-12-03"),index));
        return student;
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