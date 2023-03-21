## 检查是否安装ntpd

~~~python
rpm  -qa ntp
~~~

## 安装ntpd

~~~python
yum install  ntp   -y
~~~
## 配置ntpd

~~~python   
可以直接使用的配置文件
vim /etc/ntp.conf 
# For more information about this file, see the man pages
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).
driftfile /var/lib/ntp/drift
# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
#restrict default kod nomodify notrap nopeer noquery     
#restrict -6 default kod nomodify notrap nopeer noquery
restrict default nomodify  #添加

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1 
restrict -6 ::1
#nomdiify   #添加

# Hosts on local network are less restricted.
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server ntp1.aliyun.com   #添加
server time.nist.gov     #添加
#broadcast 192.168.1.255 autokey	# broadcast server
#broadcastclient			# broadcast client
#broadcast 224.0.1.1 autokey		# multicast server
#multicastclient 224.0.1.1		# multicast client
#manycastserver 239.255.254.254		# manycast server
#manycastclient 239.255.254.254 autokey # manycast client
# Enable public key cryptography.
#crypto
includefile /etc/ntp/crypto/pw
# Key file containing the keys and key identifiers used when operating
# with symmetric key cryptography. 
keys /etc/ntp/keys
# Specify the key identifiers which are trusted.
#trustedkey 4 8 42
# Specify the key identifier to use with the ntpdc utility.
#requestkey 8
# Specify the key identifier to use with the ntpq utility.
#controlkey 8
# Enable writing of statistics records.
#statistics clockstats cryptostats loopstats peerstats
~~~

## 启动ntpd
~~~python
/etc/init.d/ntpd start 
~~~
## 查看 
~~~python
[root@note1 ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
+ntp1.aliyun.com 10.137.38.86     2 u   53  128  377   40.931   -4.339   2.208
*utcnist2.colora .ACTS.           1 u   24  128  377  264.138  -33.896  34.067
~~~
## 查看状态
> 注意: 这个状态要等上几分钟或者一段时间才能看到 ，这个状态看到后，客户端就可以同步，
否则客户端访问的时候，就会报错

~~~python
[root@note1 ~]# ntpstat          
synchronised to NTP server (128.138.141.172) at stratum 2 
   time correct to within 175 ms
   polling server every 128 s
~~~

## 本地测试
~~~python
[root@note1 ~]# ntpdate 10.10.10.10
31 Jan 22:56:55 ntpdate[3129]: the NTP socket is in use, exiting
~~~
##  客户端同步时间
~~~python
[root@note2 ~]# ntpdate 10.10.10.10 
31 Jan 22:57:29 ntpdate[35791]: adjust time server 10.10.10.10 offset 0.023351 sec
~~~
## 定时任务
~~~python
[root@note2 ~]# crontab -l
15 1 * * * /usr/sbin/ntpdate 10.10.10.10 ; /usr/sbin/hwclock -w   &> /dev/null
~~~
