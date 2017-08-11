---
title: mybatis多数据源的支持
tags: [mybatis]
---

参考：http://blog.csdn.net/pretendcool/article/details/9343051

Mybatis框架通过sql与对象分离的方法来处理数据持久化问题。通过xml配置相应的执行sql，程序中通过xml中定义的id或命名空间+id方式获取相应的sql。

为了屏蔽掉底层数据库不同类型的语法差异问题，主要方案是针对不同数据库类型设置不同的数据库sql语法。主要包括三种方式实现mybatis对多数据库的支持。

1）方式一：

为不同的数据库的sql语法配置不同的映射xml文件。如User_mysql.xml和User_oracle.mxl文件等。在配置文件中利用通配符等方式根据不同的数据库类型加载不同的xml文件。

如属性文件中设置dbType来指定数据库类型。通过该属性值加载不同的映射文件。

2）方式二：

使用框架特性，mybatis3.1.1起，本身可以支持多数据库。

查看mybatis.xml文件（jar包中提供），在映射文件中的databaseId的值要与属性中的value相同，否则会匹配不到数据库类型。

```
<databaseIdProvider type="DB_VENDOR">  
  <property name="SQL value="sqlserver"/>  
  <property name="DB2" value="db2"/>          
  <property name="Oracle" value="oracle" />    
  <property name="Adaptive Server Enterprise" value="sybase"/>     
  <property name="MySQL" value="mysql" />  
</databaseIdProvider>
```

映射文件语法

```
<?xml version="1.0" encoding="UTF-8" ?>    
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">   
<mapper namespace="com.boco.iwms.base.dao.BasicSqlDao">  
    <select id="getCountOfSql" resultType="int" useCache="false" statementType="STATEMENT" timeout="5000" databaseId="mysql">  
        <![CDATA[ 
            SELECT COUNT(*) FROM user 
        ]]>  
    </select>  
      
    <select id="getCountOfSql" resultType="int" useCache="false" statementType="STATEMENT" timeout="5000" databaseId="oracle">  
        <![CDATA[ 
            SELECT COUNT(*) FROM user 
        ]]>  
    </select>      
</mapper>  
```

注：databaseId中指定数据类型，但一个映射文件会包含所支持数据库类型的所有语句，映射文件会比较大。

3）方式三：

局部扩充语法，只针对有必要重写的sql语法才创建，针对通用的sql语句不需要重复创建多个。

下面列举了mysql和oracle获取表的schema语法的差异，在select标签的id属性中后缀指定了数据库类型

```
//多个数据库都支持的sql语句
<select id="getById" parameterType="java.lang.Long" resultMap="BpmBusLink">
    SELECT <include refid="columns"/>
    FROM BPM_BUS_LINK
    WHERE
    BUS_ID=#{busId}
</select>

//不同数据库有不同的sql语句，通过后缀来区分
<select id="isExsitPartition_mysql" parameterType="java.util.Map" resultType="Long"> 
    select count(*)from information_schema.partitions  where table_schema = schema() and table_name='BPM_BUS_LINK' and partition_name = #{partitionName};
</select>
<select id="isExsitPartition_oracle" parameterType="java.util.Map" resultType="Long"> 
    select count(*) from user_tab_partitions where table_name = 'BPM_BUS_LINK' and partition_name = #{partitionName}
</select>
```

程序调用时，根据不同的数据库类型，添加不同的后缀，从而定位不同的sql。

```
//该功能针对不同的数据库，sql语法都相同，因此无需设置数据库类型后缀
public E getById(PK primaryKey)
{
    String getStatement= "getById";
    E object = (E)getSqlSessionTemplate().selectOne(getStatement, primaryKey);
    
    return object;
}

//程序已知该功能的执行，针对不同的数据库有不同的语法，
//因此应用程序在处理时需要先确定待执行的sql，设置不同数据库的后缀
public void createTablePartition(String tableName) {
    if(!BpmBusLink.isSupportPartition()) return;
    
    String partitionName ="P_"+ tableName.toUpperCase();
    if(partitions.contains(partitionName)) return;
    
    try {
        Map<String, String> map = new HashMap<String, String>();
        map.put("partitionName", partitionName);
        map.put("tableName", tableName);  
        
        Long count = (Long) getOne("isExsitPartition_"+this.getDbType(),map);
        if(count == 0) update("createPartition_"+this.getDbType(),map);
        partitions.add(partitionName);
    } catch (Exception e){
        e.printStackTrace();
        throw new RuntimeException("数据库类型："+this.getDbType()+" BPM_BUS_LINK 表，操作分区失败！");
    }
}
```