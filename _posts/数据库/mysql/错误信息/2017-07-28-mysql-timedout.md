---
title: mysql Connection has timed out
tags: [database]
---

An attempt by a client to checkout a Connection has timed out错误

1、错误原因

当连接池用完时客户端调用getConnection后等待获取新连接的时间，超时后将抛出SQLException。c3p0.checkoutTimeout=10000，如设为0则无限期等待。单位毫秒，默认为0 。

2、根本原因

池中的连接已经被全部使用完了，达到了最大maxconn，从而导致这个错误。

3、为什么池中连接用完？

1）连接池分配太小；

2）忘记close conn，或者异常了而没有close conn,没有释放conn从池中。

4、解决方法：

1）适当加大maxPoolSize和minPoolSize，可以大大缓解这种情况，更改c3p0.maxPoolSize和c3p0.minPoolSize。

2）检测代码 close conn，看代码是否释放conn，防止连接泄漏。

3）自动超时回收Connection强烈推荐，c3p0.unreturnedConnectionTimeout=25（单位秒）

注：为0的时候要求所有的Connection在应用程序中必须关闭。如果不为0，则强制在设定的时间到达后回收Connection，所以必须小心设置，保证在回收之前所有数据库操作都能够完成。这种限制减少Connection未关闭情况的不是很适用。为0不对connection进行回收，即使它并没有关闭。

4）配置超时自动断开conn（推荐），c3p0.maxIdleTimeExcessConnections=20，c3p0.maxConnectionAge=20（单位秒）

注：配置连接的生存时间，超过这个时间的连接将由连接池自动断开丢弃掉。当然正在使用的连接不会马上断开，而是等待它close再断开。配置为0的时候则不会对连接的生存时间进行限制。

5）c3p0.max_statements 设置成0（关闭缓存）

网上的说法：c3p0在同时关闭statement和connection的时候，或者关闭他们之间的时间很短的时候，有时候connection并没有被关闭，因为有些preparedstatement还在被cached住。这样就会有很多connection并没有真正的被关闭，连接池的连接都给耗尽了，就会产生上面的异常。解决的方案就是把缓存关闭也就是把c3p0.max_statements设置成0，这样就不会有缓存的preparedstatement。

6）c3p0.checkoutTimeout将等待超时连接时间调大。

5、c3p0的设置

maxIdleTime：60秒，C3P0默认不会close掉不用的连接池，而是将其回收到可用连接池中，这样会导致，连接数越来越大。单位是秒，maxIdleTime表示idle状态的connection能存活的最大时间。

idleConnectionTestPeriod：当数据库重启后或者由于某种原因进程被杀掉后，C3P0不会自动重新初始化数据库连接池，当新的请求需要访问数据库的时候，此时会报错误（因为连接失效），同时刷新数据库连接池，丢弃掉已经失效的连接，当第二个请求到来时恢复正常。

注：C3P0目前没有提供当获取已建立连接失败后重试次数的参数，只有获取新连接失败后重试次数的参数（acquireRetryAttempts【默认为30】）

。要解决此问题，可以通过设置idleConnectionTestPeriod【默认为0，表示不检查 】参数折中解决，该参数的作用是设置系统自动检查连接池中连接是否正常的一个频率参数，时间单位是秒 。

6、mysql查看连接数

1）查看连接进程

MySQL> show processlist; 

2）mysql连接的数量

show status like 'Aborted_clients'

Aborted_clients 由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。

Aborted_connects 尝试已经失败的MySQL服务器的连接的次数。

Connections 试图连接MySQL服务器的次数。

3）mysql配置的最大连接

show variables like '%max_connections%'--最大连接数。

4）查看表是否死锁

show global status like "table_locks%"--检查表是否被死锁。
