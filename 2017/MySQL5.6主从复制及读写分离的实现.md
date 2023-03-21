## MySQL5.6主从复制及读写分离的实现
>MySQL 5.6 基于GTID的主从复制及使用amoeba配置实现读写分离**

## amoeba 简介
Amoeba(变形虫)项目,该开源框架于2008年开始发布一款 Amoeba forMysql软件。这个软件致力于MySQL的分布式数据库前端代理层，它主要在应用层访问MySQL的时候充当SQL路由功能，专注于分布式数据库代理层（Database Proxy）开发。座落与 Client、DB Server(s)之间,对客户端透明。具有负载均衡、高可用性、SQL 过滤、读写分离、可路由相关的到目标数据库、可并发请求多台数据库合并结果。通过Amoeba你能够完成多数据源的高可用、负载均衡、数据切片的功能，目前Amoeba已在很多企业的生产线上面使用
> Amoeba优点：

1.降低费用，简单易用
2.提高系统整体可用性
3.易于扩展处理能力和系统规模
4.可以直接实现读写分离及负载均衡的效果，而不用修改代码
> Amoeba 缺点：

1.不支持事务与存储过程
2.暂不支持分库分表，amoeba目前只做到分数据库实例
3.不适合从amoeba导入数据的场景或者对大数据量查询的query并不合适（比如一次请求返回10w以上的甚至更多的数据场合）

## MySQL GTID
Mysql 5.6的新特性之一，加入了全局事务性ID(GTID:GlobalTransactions Identifier)来强化数据库的主备一致性，故障恢复，以及容错能力;也使得复制功能的配置、监控及管理变得更加易于实现，且更加健壮
## MySQL主从配置
>环境介绍：
  
```python 
[root@master~]# 192.168.17.15
[root@slave~]# 192.168.17.32
```
>安装mysql服务器在俩台主机上

```python
tar xfmysql-5.6.13-linux-glibc2.5-x86_64.tar.gz -C /usr/local
cd /usr/local/
ln -sv mysql-5.6.13-linux-glibc2.5-x86_64/mysql
cd mysql
chown –R root.msyql *
cp support-files/mysql.server/etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
echo"PATH=/usr/local/mysql/bin:$PATH" > /etc/profile
. /etc/profile
cat /etc/my.cnf | egrep -v "^#"> /root/my.cnf
cp /root/my.cnf /etc/
```
>主/etc/my.cnf的配置文件总汇
 
```python
[client]
port            = 3306
socket          = /tmp/mysql.sock
[mysqld]
port            = 3306
socket          = /tmp/mysql.sock
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 256
sort_buffer_size = 1M
read_buffer_size = 1M
read_rnd_buffer_size = 4M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size= 16M
thread_concurrency = 8
datadir=/mydata/data
innodb_file_per_table = 1
log-bin=/binlog/mysql-bin
binlog_format=row
log-slave-updates=true
gtid-mode=on
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-workers=2
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
report-port=3306
report-host=192.168.17.32
server-id       = 20
[mysqldump]
quick
max_allowed_packet = 16M
[mysql]
no-auto-rehash
[myisamchk]
key_buffer_size = 128M
sort_buffer_size = 128M
read_buffer = 2M
write_buffer = 2M
[mysqlhotcopy]
interactive-timeout
```
>初始化mysql并启动

```python
chown -R mysql.mysql  /mydata/data/
chown -R mysql.mysql  /binlog/
./scripts/mysql_install_db --user=mysql--datadir=/mydata/data/
service mysqld start
```  
>slave服务器安装mysql与master一样 ，但在slave服务器的/etc/my.cnf配置文件中有俩个参数需要更改一下与master服务器不同

```python
server-id  =  20 
report-host = 192.168.17.32 
```
> 在master服务器上创建slave复制用户并测试连接

```python
mysql> grant replication client,replication slave on *.* to 'slave'@'192.168.%.%' identified by 'budongshu';
mysql> flush privileges;
[root@slave mysql]# mysql -uslave-pbudongshu -h 192.168.17.15                                  #成功连接
```
> 启动从节点的复制线程

```python
[root@slave mysql]# mysql
mysql> change master tomaster_host='192.168.17.15',
   -> master_user='slave',
   -> master_password='budongshu',
   -> master_auto_position=1;
mysql> start slave;
mysql> show slave status \G
        Slave_IO_Running: Yes
Slave_SQL_Running:Yes
```
*yes代表启动成功,必须要俩个全部都是yes*
> 在master服务器创建数据库查看slave服务器是否更新

```python
[root@master ~]# mysql -e 'create databasebds1'
[root@slave mysql]# mysql -e 'showdatabases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| bds                |
| bds1               |                                                                        #slave服务器同步正常
| mysql              |
| performance_schema |
| test               |
+--------------------+
mysql> show processlist;                                                                       #查看gtid进程
+----+-------------+-----------+------+---------+---------+-----------------------------------------------------------------------------+------------------+
| Id | User        | Host      | db  | Command | Time    | State                                                                      | Info             |
+----+-------------+-----------+------+---------+---------+-----------------------------------------------------------------------------+------------------+
|  4| system user |           | NULL |Connect |     395 | Waiting for master tosend event                                           | NULL             |
|  5| system user |           | NULL |Connect |     225 | Slave has read allrelay log; waiting for the slave I/O thread to update it | NULL             |
|  6| system user |           | NULL |Connect | -252924 | Waiting for an event from Coordinator                                       | NULL             |
|  7| system user |           | NULL |Connect | -253320 | Waiting for an event from Coordinator                                       | NULL             |
|  9| root        | localhost | NULL |Query   |       0 | init                                                                        |show processlist |
+----+-------------+-----------+------+---------+---------+-----------------------------------------------------------------------------+------------------+
```
## 读写分离配置
基于前面做的mysql主从架构，然后在前端加一台服务器，用于实现mysql的读分离，
>安装jdk    (bin结尾的包，rpm包都可以,可以去oracle官网下载)

```python
[root@amoeba ~]# chmod +x  jdk-6u31-linux-x64-rpm.bin
[root@amoeba ~]#. /jdk-6u31-linux-x64-rpm.bin
[root@amoeba ~]# vim /etc/profile.d/java.sh
export JAVA_HOME=/usr/java/latest
export PATH=$JAVA_HOME/bin:$PATH
[root@amoeba ~]#. /etc/profile.d/java.sh
[root@amoeba ~]# java -version
java version "1.6.0_31"
Java(TM) SE Runtime Environment (build1.6.0_31-b04)
Java HotSpot(TM) 64-Bit Server VM (build20.6-b01, mixed mode)
```
> 安装amoeba

```python
[root@amoeba ~]# mkdir /usr/local/amoeba
[root@amoeba ~]# tar xf  amoeba-mysql-binary-2.2.0.tar.gz -C /usr/local/
[root@amoeba ~]# vi/etc/profile.d/amoeba.sh
export AMOEBA_HOME=/usr/local/amoeba
export PATH=$AMOEBA_HOME/bin:$PATH
[root@amoeba ~]# .  /etc/profile.d/amoeba.sh
[root@amoeba ~]# amoeba                                                                          #出现下边信息代表安装成功
amoeba start|stop
```
>授权MySQL用户，用于实现前端amoeba连接

```python
mysql> grant all on *.* to'amoeba'@'192.168.%.%' identified by 'amoebapass';
mysql> flush privileges;
```
>配置amoeba

```python
[root@amoeba ~]# cd /usr/local/amoeba/conf/
amoeba.xml                                                                                       #定义管理信息与读写分离
dbServers.xml                                                                                    #定义后端服务器的配置
```
> 配置文件dbServers.xml介绍

```xml
[root@amoeba conf]# vi dbServers.xml
<?xml version="1.0"encoding="gbk"?>
<!DOCTYPE amoeba:dbServers SYSTEM"dbserver.dtd">
<amoeba:dbServersxmlns:amoeba="http://amoeba.meidusa.com/">
                <!--
                        Each dbServer needs tobe configured into a Pool,
                        If you need toconfigure multiple dbServer with load balancing that can be simplified by thefollowing configuration:
                         add attribute with name virtual ="true" in dbServer, but the configuration does not allow the elementwith name factoryConfig
                         such as 'multiPool'dbServer
                -->
       <dbServer name="abstractServer" abstractive="true">
                <factoryConfigclass="com.meidusa.amoeba.mysql.net.MysqlServerConnectionFactory">
                        <propertyname="manager">${defaultManager}</property>
                        <propertyname="sendBufferSize">64</property>
                        <propertyname="receiveBufferSize">128</property>
                        <!-- mysql port-->
                        <propertyname="port">3306</property>                    #连接后端mysql服务器的端口
                        <!-- mysql schema-->
                       <propertyname="schema">test</property>                   #连接后端mysql服务器的默认库
                        <!-- mysql user-->
                        <propertyname="user">amoeba</property>                  #连接后端mysql服务器的用户名
                        <!--  mysql password -->                                #把password最后的注释(-->)这个符号去掉跟上边一样
                        <propertyname="password">amoebapass</propert>           #连接后端mysql服务器的密码
                </factoryConfig>
                <poolConfigclass="com.meidusa.amoeba.net.poolable.PoolableObjectPool">
                        <propertyname="maxActive">500</property>
                        <propertyname="maxIdle">500</property>
                        <propertyname="minIdle">10</property>
                        <propertyname="minEvictableIdleTimeMillis">600000</property>
                        <propertyname="timeBetweenEvictionRunsMillis">600000</property>
                        <propertyname="testOnBorrow">true</property>
                        <propertyname="testOnReturn">true</property>
                        <propertyname="testWhileIdle">true</property>
                </poolConfig>
       </dbServer>
       <dbServer name="master" parent="abstractServer">                     #定义master服务器的节点 name可以自定义
                <factoryConfig>
                        <!-- mysql ip -->
                        <propertyname="ipAddress">192.168.17.15</property>  #定义master服务器的IP地址
                </factoryConfig>
       </dbServer>
       <dbServer name="slave" parent="abstractServer">                      #定义slave服务器的节点
                <factoryConfig>
                        <!-- mysql ip -->
                        <propertyname="ipAddress">192.168.17.32</property>
                </factoryConfig>
        </dbServer>
       <dbServer name="multiPool" virtual="true">
                <poolConfigclass="com.meidusa.amoeba.server.MultipleServerPool">
                        <!-- Load balancingstrategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->    #负载均衡算法
                        <propertyname="loadbalance">1</property>                               #定义选择哪一种算法
                        <!-- Separated bycommas,such as: server1,server2,server1 -->           #定义数据库池，用于实现负载均衡，“slave“为自定义的数据库节点，可以写多个用”，”隔开
                        <property name="poolNames">slave</property
                </poolConfig>
       </dbServer>
</amoeba:dbServers>
```
----------------------------------------------------------------------------------------------------------------
> 配置文件amoeba.xml 介绍

```python
[root@amoeba conf]# vi amoeba.xml
<?xml version="1.0"encoding="gbk"?>
<!DOCTYPE amoeba:configuration SYSTEM"amoeba.dtd">
<amoeba:configurationxmlns:amoeba="http://amoeba.meidusa.com/">
       <proxy>
                <!-- service class mustimplements com.meidusa.amoeba.service.Service -->
               <servicename="Amoeba for Mysql"class="com.meidusa.amoeba.net.ServerableConnectionManager">
                        <!-- port -->
                        <propertyname="port">3306</property>                        #定义amoeba代理服务器对外监听的端口
                        <!-- bind ipAddress-->
                        <!--
                        <propertyname="ipAddress">192.168.17.11</property>          #定义amoeba代理服务器对外连接的监听ip      
                         -->
                        <propertyname="manager">${clientConnectioneManager}</property>
                        <propertyname="connectionFactory">
                                <beanclass="com.meidusa.amoeba.mysql.net.MysqlClientConnectionFactory">
                                       <property name="sendBufferSize">128</property>
                                       <property name="receiveBufferSize">64</property>
                                </bean>
                        </property>
                        <propertyname="authenticator">
                                <beanclass="com.meidusa.amoeba.mysql.server.MysqlClientAuthenticator">   #定义客户端使用的用户名和密码
                                       <property name="user">admin</property>                               
                                       <property name="password">password</property>
                                       <property name="filter">
                                               <beanclass="com.meidusa.amoeba.server.IPAccessController">
                                                       <propertyname="ipFile">${amoeba.home}/conf/access_list.conf</property>
                                               </bean>
                                       </property>
                                </bean>
                        </property>
                </service>
                <!-- server class mustimplements com.meidusa.amoeba.service.Service -->
                <service name="AmoebaMonitor Server"class="com.meidusa.amoeba.monitor.MonitorServer">
                        <!-- port -->
                        <!--  default value: random number
                        <propertyname="port">9066</property>
                        -->
                        <!-- bind ipAddress-->
                        <propertyname="ipAddress">127.0.0.1</property>
                        <propertyname="daemon">true</property>
                        <propertyname="manager">${clientConnectioneManager}</property>
                        <propertyname="connectionFactory">
                                <beanclass="com.meidusa.amoeba.monitor.net.MonitorClientConnectionFactory"></bean>
                        </property>
                </service>
                <runtimeclass="com.meidusa.amoeba.mysql.context.MysqlRuntimeContext">
                       <!-- proxy servernet IO Read thread size -->
                        <propertyname="readThreadPoolSize">20</property>
                        <!-- proxy serverclient process thread size -->
                        <propertyname="clientSideThreadPoolSize">30</property>
                        <!-- mysql serverdata packet process thread size -->
                        <propertyname="serverSideThreadPoolSize">30</property>
                        <!-- per connectioncache prepared statement size  -->
                        <propertyname="statementCacheSize">500</property>
                        <!-- query timeout(default: 60 second , TimeUnit:second) -->
                        <propertyname="queryTimeout">60</property>
                </runtime>
       </proxy>
       <!--
                Each ConnectionManager willstart as thread
                manager responsible for theConnection IO read , Death Detection
       -->
       <connectionManagerList>
                <connectionManagername="clientConnectioneManager"class="com.meidusa.amoeba.net.MultiConnectionManagerWrapper">
                        <propertyname="subManagerClassName">com.meidusa.amoeba.net.ConnectionManager</property>
                        <!--
                          default value isavaliable Processors
                        <propertyname="processors">5</property>
                         -->
                </connectionManager>
                <connectionManagername="defaultManager" class="com.meidusa.amoeba.net.MultiConnectionManagerWrapper">
                        <propertyname="subManagerClassName">com.meidusa.amoeba.net.AuthingableConnectionManager</property>
                        <!--
                          default value is avaliableProcessors
                        <propertyname="processors">5</property>
                         -->
                </connectionManager>
       </connectionManagerList>
                <!-- default using fileloader -->
       <dbServerLoader class="com.meidusa.amoeba.context.DBServerConfigFileLoader">
                <propertyname="configFile">${amoeba.home}/conf/dbServers.xml</property>
       </dbServerLoader>
       <queryRouterclass="com.meidusa.amoeba.mysql.parser.MysqlQueryRouter">
                <propertyname="ruleLoader">
                        <beanclass="com.meidusa.amoeba.route.TableRuleFileLoader">
                                <propertyname="ruleFile">${amoeba.home}/conf/rule.xml</property>
                                <propertyname="functionFile">${amoeba.home}/conf/ruleFunctionMap.xml</property>
                        </bean>
                </property>
                <propertyname="sqlFunctionFile">${amoeba.home}/conf/functionMap.xml</property>
                <propertyname="LRUMapSize">1500</property>
                <propertyname="defaultPool">master</property>      #把 <!--   -->注释去掉使其配置生效, 定义默认池，默认会在此服务器上执行
                <property name="writePool">master</property>       #定义只写服务器    
                <propertyname="readPool">slave</property>          #定义只读服务器，也可以在dbServer.xml中定义数据池的名称，实现负载均衡                                      

                <propertyname="needParse">true</property>                            
       </queryRouter>
</amoeba:configuration>
```

>启动amoeba服务并测试

```python
[root@amoeba conf]# amoeba start &                                                #后台启动amoeba
[root@amoeba conf]# ss -tunlp | grep 3306                                         #启动正常
tcp   LISTEN     0      128                   :::3306                 :::*      users:(("java",1796,52))
```
>连接amoeba代理服务器，执行插入和查询操作，分别在后端俩台服务器进行抓包，查看是否实现读写分离

```python
mysql> create database bds2;
mysql> use bds2
mysql> create table tb2 (id int ) ;
mysql> select * from tb2;
 ```
## tcpdump抓包查看检测

```python
[root@master ~]# tcpdump -i eth0 -s0 -nn -A tcp dst port 3306 and dst  host192.168.17.15
  
13:25:46.488469 IP 192.168.17.11.39787 >192.168.17.15.3306: Flags [P.], seq 171:202, ack 444, win 490, options[nop,nop,TS val 8050703 ecr 15047099], length 31
E..S4  @.@.c1.........k....z..y.............
.z...........create table tb2 (id int )                                     ###写请求在master服务器上执行
13:25:46.523028 IP 192.168.17.32.41112 >192.168.17.15.3306: Flags [.], ack 1661, win 1117, options [nop,nop,TS val41437531 ecr 15058573], length 0
E..4..@.@.y{... ........M.$....4...].   .....
.xI[....
13:25:46.523050 IP 192.168.17.11.39787 >192.168.17.15.3306: Flags [.], ack 455, win 490, options [nop,nop,TS val 8050737ecr 15058574], length 0
E..44
@.@.cO.........k....z..y......Q......
.z.1....
 
[root@slave ~]# tcpdump -i eth0 -s0 -nn -Atcp dst  port 3306 and dst  host 192.168.17.32
 
15:01:20.243577 IP 192.168.17.11.54071 >192.168.17.32.3306: Flags [.], ack 196, win 457, options [nop,nop,TS val8129871 ecr 41516665], length 0
E..4@.@.@.V}....... .7..}.      ....i...........
O.y~y
15:01:20.246139 IP 192.168.17.11.54071 >192.168.17.32.3306: Flags [P.], seq 133:155, ack 196, win 457, options[nop,nop,TS val 8129874 ecr 41516665], length 22
E..J@.@.@.Vf....... .7..}.      ....i...........
R.y~y.....select * from tb2                                                 ###读请求在slave服务器上执行
15:01:20.287625 IP 192.168.17.11.54071 >192.168.17.32.3306: Flags [.], ack 259, win 457, options [nop,nop,TS val8129915 ecr 41516669], length 0
E..4@.@.@.V{....... .7..}.      ..........}.....
{.y~}
```
*由上图可知抓包实现了读写分离的效果*
 
