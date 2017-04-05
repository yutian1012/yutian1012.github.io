---
title: mysql编码设置
tags: [database]
---

MySQL的字符集支持(Character Set Support)有两个方面：字符集(Character set)和排序方式(Collation)。

MySQL对于字符集的支持细化到四个层次: 服务器(server)，数据库(database)，数据表(table)和连接(connection)。

乱码的出现：一般是客户端、服务器、结果集、数据库的字符集不统一造成的。

注：低版本的mysql的my.init文件有多个节点，[client]、[mysql]、[mysqld]配置方式
参考：http://www.jb51.net/article/38122.htm

设置完成后需要重启mysql：net stop myslq和net start mysql

操作环境：mysql-5.6.23-winx64

参考：http://www.th7.cn/db/mysql/201412/84636.shtml

参考：http://blog.csdn.net/sin90lzc/article/details/7648439

参考：http://niutuku.com/tech/Mysql/237673.shtml

### 设置mysql的编码格式

1. 查看数据库中支持的字符集

```
show character set;
或者
show char set;
```

![](/images/database/mysql/mysql-character-set.png)

查看数据库当前状态（包括字符编码）

```
status
```

![](/images/database/mysql/mysql-character-status.png)

2. 查看系统字符集设置，包括所有的字符集设置：

``` 
show variables like 'character%';
```

![](/images/database/mysql/mysql-character.png)

1）字符集查看与理解

character_set_client：客户端使用的字符集

```
What character set is the statement in when it leaves the client?

The server takes the character_set_client system variable to be the character set in which statements are sent by the client.

character-set-client：客户端的字符集。客户端默认字符集。当客户端向服务器发送请求时，请求以该字符集进行编码。
character-set-results：结果字符集。服务器向客户端返回结果或者信息时，结果以该字符集进行编码。
在客户端，如果没有定义character-set-results，则采用character-set-client字符集作为默认的字符集。所以只需要设置character-set-client字符集。
```

character_set_connection：连接数据库的字符集设置类型，如果程序没有指明连接数据库使用的字符集类型则按照服务器端默认的字符集

```
What character set should the server translate a statement to after receiving it?

For this, the server uses the character_set_connection and collation_connection system variables. It converts statements sent by the client from character_set_client to character_set_connection (except for string literals that have an introducer such as _latin1 or _utf8). collation_connection is important for comparisons of literal strings. For comparisons of strings with column values, collation_connection does not matter because columns have their own collation, which has a higher collation precedence.
```

character_set_database：数据库服务器中某个库使用的字符集设定，如果建库时没有指明，将使用服务器安装时指定的字符集

character_set_results：为数据库给客户端返回信息时使用的字符集设定，如果没有指明，使用服务器默认的字符集

```
What character set should the server translate to before shipping result sets or error messages back to the client?

The character_set_results system variable indicates the character set in which the server returns query results to the client. This includes result data such as column values, and result metadata such as column names and error messages.
```

character_set_server：服务器安装时指定的默认字符集

```
character-set-server/default-character-set：服务器字符集，默认情况下所采用的。
character-set-database：数据库字符集。
character-set-table：数据库表字符集。
优先级依次增加。所以一般情况下只需要设置character-set-server，而在创建数据库和表时不特别指定字符集，这样统一采用character-set-server字符集。
```

character_set_system：数据库系统使用的字符集，用于数据库中对象（如表和列）的名字，也用于存储在目录表中函数的名字。其值总是等于utf8

注：从实际上可以看到，当客户端连接服务器的时候，它会将自己想要的字符集名称发给mysql服务器，然后服务器就会使用这个字符集去设置character_set_connection、character_set_client、character_set_results这三个值。如cmd是用gbk，而mysql workbench是用utf8.

注：要处理中文，则可以将character-set-server和character-set-client均设置为GB2312（设置相同的字符编码），如果要同时处理多国语言，则设置为UTF8。

2）使用cmd利用客户端连接查看

设置连接使用的字符集

```
SET NAMES 'gbk';
SET CHARACTER SET 'gbk';
```

再次查看字符集设置（show variables命令），发现下面的三个字段的字符集被修改了

```
 character_set_client     | gbk
 character_set_connection | gbk
 character_set_results    | gbk
```

注：这种方式是临时会话有效，当我们quit出mysql后，再次登录发现设置的字符集失效了。

注：当一个客户端连接时，它向服务器发送希望使用的字符集名称，服务器把变量character_set_client、character_set_results和character_set_connection设置为该字符集，即实际上服务器为使用该字符集执行一个SET NAMES操作。

也可以针对每个变量设置相应的字符

```
命令行方式设置编码，只在当前窗口中有效。
SET character_set_client = utf8; 
SET character_set_connection = utf8; 
SET character_set_database = utf8; 
SET character_set_results = utf8; 
SET character_set_server = utf8; 
```

3）字符集设置

```
[mysql]
default-character-set=utf8
```

修改：character_set_client、character_set_connection、character_set_results变量的编码值

```
[mysqld]
character-set-server=utf8
```

修改：character_set_database（创建数据库时）、character_set_server变量的编码值

3. 字符序

1）定义信息

mysql的collation大致的意思就是字符序，首先字符本来是不分大小的，那么对字符的>, = , < 操作就需要有个字符序的规则。collation做的就是这个事情，你可以对表进行字符序的设置，也可以单独对某个字段进行字符序的设置。

2）查看mysql的字符序

```
show variables like 'collation%';
```

![](/images/database/mysql/mysql-character2.png)

注：以_ci(表示大小写不敏感)，以_cs(表示大小写敏感)，以_bin(表示用编码值进行比较)。

3）在表字段上设置字符序

可在定义表字段时设置字符序

```
`content` varchar(255) NOT NULL COLLATE utf8_bin,
```

注：如果对字符大小敏感的话，最好将数据库中默认的utf8_general_ci设置为utf8_bin。

设置方式：修改my.ini文件，在[mysqld]下添加

```
collation-server=utf8_general_ci
```

4. 实例

1）存储字符（字段/表）与 发送字符（客户端发送sql）的转换

2）character_set_client和character_set_connection、character_set_results不一样

3）character_set_results参与的转换
参考：http://www.server110.com/mysql/201403/7139.html

4）程序实例操作中文乱码
参考：http://blog.sina.com.cn/s/blog_81547cad01014x0v.html

5. 数据库、数据表、字段级别的编码设置
