---
title: centos7编译安装zabbix3.0
date: 2016-09-02 20:26:20
categories: zabbix
tags: zabbix
---
## Centos7 编译zabbix3.0

## 系统环境
```
[root@localhost ~]# uname -a 
Linux localhost.localdomain 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 


```
## 安装yum源
```
[root@localhost ~]# wget -P /etc/yum.repos.d http://mirrors.aliyun.com/repo/Centos-7.repo
[root@localhost ~]# yum install -y epel*
```

## lamp环境
### 说明
>  zabbix3.0不能直接在centos6上以rpm包的形式安装,所以系统环境是centos7的,如果要在centos6上
安装采用编译的形式安装,这里centos7,我也使用编译安装的方式,同时zabbix3.0要求php的版本比较高
注意php的版本


### centos7的mysql用的是mariadb
```
[root@localhost ~]# yum search mysql |grep mariadb
Repository base is listed more than once in the configuration
Repository updates is listed more than once in the configuration
Repository extras is listed more than once in the configuration
Repository centosplus is listed more than once in the configuration
mariadb.x86_64 : A community developed branch of MySQL
mariadb-devel.i686 : Files for development of MariaDB/MySQL applications
mariadb-devel.x86_64 : Files for development of MariaDB/MySQL applications
mariadb-libs.i686 : The shared libraries required for MariaDB/MySQL clients
mariadb-libs.x86_64 : The shared libraries required for MariaDB/MySQL clients

```

### 安装lamp
```
[root@localhost ~]# yum -y install mariadb mariadb-server php php-mysql httpd mysql-devel libxml2-devel \
libcurl-devel net-snmp-devel  libxml2-devel libXpm php-bcmath php-gd php-mbstring php-xml t1lib
```

### 软件版本
![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/1.png)
### 启动服务 
```
启动mariadb
[root@localhost ~]# systemctl enable mariadb 
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@localhost ~]# systemctl start mariadb 

```
![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/2.png)

## 配置设置
### 数据库root密码和初始化安全设置
![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/3.png)
- 第一个红框表示: 现在的默认的root用户的密码,直接回车就好.因为默认root用户没有密码 
- 第二个红框表示: 为root设置一个密码

![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/4.png) 

- 第一个 是否移除匿名用户 
- 第二个 是否允许root远程访问
- 第三个 移除test测试数据库
- 第四个 重新加载


### 建立zabbix数据库和授权访问
```
[root@localhost ~]# mysql -uroot -pmysql  -e "create database zabbix default character set utf8 collate utf8_bin;" 
[root@localhost ~]# mysql -uroot -pmysql  -e "grant all on zabbix.* to 'zabbix'@localhost identified by 'zabbix';" 
```

### 测试配置 
![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/5.png) 　

### 启动服务
```
systemctl start httpd 
```
**lamp 安装完毕**
## 安装编译zabbix3.0
### 下载加压
```
[root@localhost ~]# wget http://jaist.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.0.4/zabbix-3.0.4.tar.gz
[root@localhost ~]# tar xf zabbix-3.0.4.tar.gz  
[root@localhost ~]# cd /root/zabbix-3.0.4/database/mysql 

```
### 拷贝sql到zabbix数据库
```
[root@localhost mysql]# mysql -uzabbix  -pzabbix  zabbix < schema.sql   
[root@localhost mysql]# mysql -uzabbix  -pzabbix  zabbix < images.sql 
[root@localhost mysql]# mysql -uzabbix  -pzabbix  zabbix < data.sql

```
### 编译zabbix3.0
```
[root@localhost zabbix-3.0.4]# ./configure  --prefix=/usr/local/zabbix --enable-server --enable-agent --with-mysql --with-net-snmp --with-libcurl --enable-proxy --with-libxml2
[root@localhost zabbix-3.0.4]# make && make install 

```
### 修改zabbix配置文件
```
[root@localhost ~]# grep -v "^#" /usr/local/zabbix/etc/zabbix_server.conf  |grep -v "^$"
LogFile=/tmp/zabbix_server.log
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
DBPort=3306
StartPollers=5
ListenIP=192.168.199.211
Timeout=4
AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts
ExternalScripts=/usr/local/zabbix/share/zabbix/externalscripts
LogSlowQueries=3000

```
### 开机自启动
```
[root@localhost ~]# sed -i '$a /usr/local/zabbix/sbin/zabbix_server  -c /usr/local/zabbix/etc/zabbix_server.conf' /etc/rc.local 
[root@localhost ~]# chmod +x /etc/rc.d/rc.local
```
### 拷贝页面文件到站点目录
```
[root@localhost ~]# mkdir /data/webroot -pv     #把zabbix站点放到此目录
[root@localhost ~]# cp -r /root/zabbix-3.0.4/frontends/php/ /data/webroot/zabbix
重启httpd
[root@localhost zabbix]# service httpd restart 

```
### 修改httpd配置文件
```
[root@localhost ~]# cd /etc/httpd/conf.d/
[root@localhost conf.d]# cat zabbix.conf 
<VirtualHost 192.168.199.211:80>
        DocumentRoot /data/webroot/zabbix
        ServerName bds.zabbix.com 
        DirectoryIndex index.php index.html 
        ErrorLog  logs/zabbix-error.log
        CustomLog logs/zabbix-access.log common
<Directory "/data/webroot/zabbix">
    Options None
    AllowOverride None
    Require ip 192.168.0     #访问控制ip
    Require all granted
   
    <IfModule mod_php5.c>
        php_value max_execution_time 300
        php_value memory_limit 128M
        php_value post_max_size 16M
        php_value upload_max_filesize 2M
        php_value max_input_time 300
        php_value always_populate_raw_post_data -1
       php_value date.timezone Asia/Chongqing
     </IfModule>
</Directory>
</VirtualHost>
再次重启httpd
[root@localhost zabbix]# service httpd restart 

```
## web界面安装
> 在windows主机做本地hosts解析
路径: C:\Windows\System32\drivers\etc 修改hosts文件
在最后加一条 192.168.199.211 bds.zabbix.com  ###请换成你自己的ip地址 


![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/11.png)

![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/1111.png)

![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/12.png)

![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/13.png) 　

![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/14.png)  　 　

**按照提示download下载配置文件,然后上传到/data/webroot/zabbix/conf/ 目录下 
再把zabbix.conf.php 里面
$ZBX_SERVER      = 'localhost'; 
改成自己的ip地址 
$ZBX_SERVER      = '192.168.199.211';
注意: 需要跟你的zabbix_server.conf配置文件中,配置的主机地址一致 
最后刷新页面**   


　

![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/15.png)


![](http://7xnvvj.com1.z0.glb.clouddn.com//zabbix3.0/16.png) 　
