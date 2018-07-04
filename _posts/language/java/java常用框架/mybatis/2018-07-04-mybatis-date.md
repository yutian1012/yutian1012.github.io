---
title: mybatis处理日期的问题
tags: [mybatis]
---

1）保存对象时设置日期抛出异常信息

```
Caused by: com.mysql.jdbc.MysqlDataTruncation: Data truncation: Incorrect date value: '' for column 'createDate' at row 1
```

2）实体类

```
@Setter
@Getter
@NoArgsConstructor
@Accessors(chain=true)
public class PageLog extends PageModel{
    private String id;
    private String url;
    private String note;
    private PageLogStatusEnum status;
    private PageLogExceptionEnum exceptionType;//异常类型
    private Date createDate;
}
```

注：实体类的createDate字段是Date类型的

3）mapper映射文件

```
<insert id="add" parameterType="PageLog">
    insert into page_log (id,url,note,status,exceptionType,createDate) values(#{id},#{url},#{note},#{status},#{exceptionType},#{createDate});
</insert>
```

数据表

```
create table page_log(
    id varchar(64) not null primary key comment '日志主键字段',
    url varchar(1024) not null comment '爬取地址',
    note text comment '异常信息',
    status varchar(32) not null comment '状态',
    exceptionType varchar(32) comment '异常类型',
    createDate date not null comment '创建日期'
) comment '爬虫日志表';
```

注：使用的是mysql数据库的date类型

4）测试代码

```
private SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
@Test
public void testAdd() throws ParseException {
    String id=UUID.randomUUID().toString();
    PageLog pageLog=new PageLog();
    pageLog.setId(id);
    pageLog.setNote("测试");
    pageLog.setStatus(PageLogStatusEnum.PROCESSING);
    pageLog.setUrl("url");
    //Date date=sdf.parse("2018-07-04");
    pageLog.setCreateDate(new Date());
    
    pageLogDao.add(pageLog);
    
    PageLog tmp=pageLogDao.getById(id);
    assertNotNull(tmp);
}
```

直接保存数据出现上面的异常信息，但使用SimpleDateFromat转换的数据却没有异常信息。

5）修改方法

方式一：使用SimpleDateFromat转换

方式二：在mapper文件中添加jdbcType=DATE

注：怀疑是由于内部转换时，数值长度问题引起的。日期本身是long类型的数据。