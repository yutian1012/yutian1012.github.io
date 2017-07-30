---
title: jdbc方式连接并访问数据库
tags: [datasource]
---

参考：http://www.cnblogs.com/wang-meng/p/5373438.html

JDBC其实一套规范（接口）数据库厂商需要实现此接口(实现类)，称为数据库驱动。

jdbc的作用：可以和数据库创建链接；发送sql语句；接收返回值，处理结果。

### API

1）DriverManager 类:

管理一组JDBC驱动程序的基本服务。registerDriver(Driver)注册驱动方法， getConnection方法获取连接。

```
Class.forName("com.mysql.jdbc.Driver");
```

注：运行该段代码会自动调用DriverManager类的registerDriver方法，可以查看Driver类的源码实现。

2）Connection 接口

获取连接：

```
Connection conn=DriverManager.getConnection("jdbc:mysql://localhost:3306/database", "root", "123456");
```

常用方法

```
Statement createStatement();//创建Statement -语句执行者
PreparedStatement prepareStatement(String sql);//创建一个预编译的语句执行对象
//创建一个CallableStatement对象来调用数据库存储过程。
CallableStatement prepareCall(String sql)
```

3）Statement 接口

该类容易产生sql注入，使用PreparedStatement代替。

获取Statement对象

```
Statement st=conn.createStatement();
```

常用方法：

```
ResultSet executeQuery(String sql)//执行查询语句,返回一个集合
int executeUpdate(String sql)//执行更新 插入 删除语句,返回影响行数

//:执行给定的 SQL 语句，该语句可能返回多个结果
//若返回true，执行是的查询语句，调用 getResultSet 获取查询结果
//若返回false，执行的是更新/插入/删除语句，调用 getUpdateCount 获取影响的行数
boolean execute(sql)
```

注：execute是一个综合方法。

4）ResultSet 接口

获取查询结果集

```
ResultSet rs=st.executeQuery(sql);
```

常用方法

```
boolean next()//判断是否有下一条记录,并且移动到下一行

//参数设置有两种方式
//方式一：字段名称，字符串
//方式二：第几列，下标从1开始
getXXX(参数)//获取内容，如getInt(1)
```

### 实例

1）建表语句

```
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `name` varchar(32) NOT NULL,
  `email` varchar(32) DEFAULT NULL,
  `passwd` varchar(32) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

2）创建JDBC连接的工具类

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ResourceBundle;

public class JDBCUtil {
    static final String DRIVERNAME;
    static final String URL;
    static final String USERNAME;
    static final String PASSWORD;
    
    static{
        ResourceBundle bundle=ResourceBundle.getBundle("jdbc");
        DRIVERNAME=bundle.getString("drivername");
        URL=bundle.getString("url");
        USERNAME=bundle.getString("user");
        PASSWORD=bundle.getString("password");
    }
    
    /*
     * 如果jdbc的jar包中提供了SPI查找Driver机制，这段静态代码块可以省略了
     * static{
        try {
            Class.forName(DRIVERNAME);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }*/
    
    //获取链接
    public static Connection getConnection() throws SQLException{
        return DriverManager.getConnection(URL,USERNAME,PASSWORD);
    }
    
    //释放资源
    public static void closeResource(Connection conn,Statement st,ResultSet rs){
        if (rs!=null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (st!=null) {
            try {
                st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (conn!=null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
    }
}
```

ResourceBundle类加载classpath下的jdbc.properties获取数据库连接配置信息。

jdbc.properties文件内容

```
drivername=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=UTC
user=root
password=root
```

注：jdbc连接可以设置连接url参数，url参数的设置方式与http浏览器设置参数类似。

3）创建测试

```
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

//数据库操作测试
public class CRUDDemo {
    public static void main(String[] args) {
        //插入
        insert(1,"赵四","123","zhaosi@163.com");
        //更新
        updateByName("赵四","尼古拉斯.赵四");
        //获取
        getByName("尼古拉斯.赵四");
        //删除
        deleteByName("尼古拉斯.赵四");
    }

    private static void deleteByName(String string) {
        //模版
        Connection conn=null;
        PreparedStatement st=null;
        ResultSet rs=null;
        
        try {
            //获取链接
            conn=JDBCUtil.getConnection();
            //编写sql
            String sql="delete from user where name =?";
            //获取预编译执行者
            st=conn.prepareStatement(sql);
            //设置参数
            st.setString(1, string);
            //执行sql
            int i = st.executeUpdate();
            //处理结果
            if (i>0) {
                System.out.println("成功");
            }else{
                System.out.println("失败");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            //释放资源
            JDBCUtil.closeResource(conn, st, rs);
        }
        
    }

    private static void getByName(String string) {

        Connection conn=null;
        PreparedStatement st=null;
        ResultSet rs=null;
        
        try {
            conn=JDBCUtil.getConnection();
            String sql="select * from user where name=? limit 1";
            st=conn.prepareStatement(sql);
            
            st.setString(1, string);
            rs=st.executeQuery();
            if (rs.next()) {
                System.out.println(rs.getInt(1)+":"+rs.getString(2)+":"+rs.getObject(3)+":"+rs.getObject(4));
            }
        } catch (Exception e) {
            // TODO: handle exception
            e.printStackTrace();
        }finally{
            
            JDBCUtil.closeResource(conn, st, rs);
        }
        
    }

    private static void updateByName(String oldName, String newName) {
        Connection conn=null;
        PreparedStatement st=null;
        ResultSet rs=null;
        
        try {
            conn=JDBCUtil.getConnection();
            String sql="update user set name = ? where name = ?";
            st=conn.prepareStatement(sql);
            st.setString(1, newName);
            st.setString(2, oldName);
            
            int i=st.executeUpdate();
            //处理结果
            if (i>0) {
                System.out.println("成功");
            }else{
                System.out.println("失败");
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            JDBCUtil.closeResource(conn, st, rs);
        }
        
        
    }

    private static void insert(int id,String username, String password, String email) {
        Connection conn=null;
        PreparedStatement st=null;
        ResultSet rs=null;
        
        try {
            //获取链接
            conn=JDBCUtil.getConnection();
            //编写sql
            String sql="insert into user (id,name,passwd,email) values(?,?,?,?)";
            //获取预编译语句执行者
            st=conn.prepareStatement(sql);
            //设置参数
            st.setInt(1, id);
            st.setString(2, username);
            st.setString(3, password);
            st.setString(4, email);
            //执行sql
            int i=st.executeUpdate();
            //处理结果
            if (i>0) {
                System.out.println("成功");
            }else{
                System.out.println("失败");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            JDBCUtil.closeResource(conn, st, rs);
        }
    }

}
```