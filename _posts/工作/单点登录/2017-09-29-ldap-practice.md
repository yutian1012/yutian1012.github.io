---
title: LDAP服务--实例
tags: [cas]
---

### 实例创建LDAP数据条目

参考：http://jianshi-dlw.iteye.com/blog/1557846

参考：http://www.linuxidc.com/Linux/2013-07/88082.htm

公司域名未：maxcrc.com，下面设置了若干个部门sales，resources，develop等，每个部门中又包含若个人员。

1）修改根条目的限制信息（OPENLDAP_HOME目录下）

在slapd.conf配置文件中存在根条目相关的限制信息

```
database    bdb
suffix      "dc=maxcrc,dc=com"
rootdn      "cn=Manager,dc=maxcrc,dc=com"
```

注：这里修改的条目并不会在LDAP服务器中出现，而只是在LDAP库中创建数据时要符合这里的限制，即根的条目信息为这里配置的信息。

2）启动LDAP服务（OPENLDAP_HOME目录下）

```
slapd -d 1 -f slapd.conf
```

注：在cmd中运行，-d表示debug，后面的数值1，表示level等级。-f参数指定配置文件。

![](/images/work/cas/ldap/openldap-server-start.png)

3）创建数据条目（根）--需要满足slapd.conf文件中的限制

创建root.ldif（LDAP Data Interchanged Format）文件。

```
dn: o=maxcrc,dc=maxcrc,dc=com
objectclass: top
objectclass: organization
o: maxcrc
```

执行命令（OPENLDAP_HOME\ClientTools目录下）

```
ldapmodify -a -x -D "cn=Manager,dc=maxcrc,dc=com" -w secret -f D:/ldap/root.ldif
```

注：-a表示添加，-D指定LDAP的管理区，-w指定密码，-f指定ldif文件，本地测试不需要加host信息，如果需要添加可以使用-h参数。cn=Manager是认证的用户信息。

![](/images/work/cas/ldap/ladpadd-root.png)

删除条目

```
ldapdelete -x -D "cn=Manager,dc=maxcrc,dc=com" -w secret "o=maxcrc,dc=maxcrc,dc=com"
```

注：-D指定LDAP的管理区，最后的内容为删除的信息。

查询条目信息

```
ldapsearch -x -D "cn=Manager,dc=maxcrc,dc=com" -w secret -s '(objectclass=*)'
```



4）创建数据条目（枝）

创建branch.ldif文件。

```
dn: ou=Managers,dc=maxcrc,dc=com
objectClass: top
objectClass: organizationalUnit
ou: Managers
```

执行命令（OPENLDAP_HOME\ClientTools目录下）

```
ldapmodify -a -x -D "cn=AcmeManager,o=Acme,c=US" -w secret -f D:/ldap/branch.ldif
```