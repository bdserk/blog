## 安装yum依赖
yum install curl-devel bzip2-devel python-devel openssl-devel readline-devel perl-ExtUtils-Embed libxml2-devel openldap-devel pam pam-devel perl-devel apr-devel libevent-devel libyaml libyaml-devel libedit-devel libffi-devel bison flex -y

yum -y install ncurses ncurses-devel libaio-devel gmp gmp-devel mpfr mpfr-devel libmpc libmpc-devel zlib-devel net-tools cmake openssl openssl-devel gcc-c++ 

## 编译cmake                                   
cd /root 
wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz --no-check-certificate 
tar xf cmake-3.5.2.tar.gz 
cd cmake-3.5.2 
./bootstrap 
make -j 96 && make install && hash -r &&  /usr/local/bin/cmake --version
## 编译安装kaezip
### 下载kaezip
cd /root
wget https://github.com/kunpengcompute/KAEzip/archive/refs/tags/v1.3.11.tar.gz
 tar xf v1.3.11.tar.gz
cd KAEzip-1.3.11/
### 下载zlib
wget https://nchc.dl.sourceforge.net/project/libpng/zlib/1.2.11/zlib-1.2.11.tar.gz
#把zlib拷贝到kae zip 相应的目录中
cp  zlib-1.2.11.tar.gz KAEzip-1.3.11/open_source/        

### 下载KAEdriver
cd /opt/
wget https://github.com/kunpengcompute/KAEdriver/archive/refs/tags/v1.3.11.tar.gz
tar xf v1.3.11.tar.gz
cd KAEdriver-1.3.11/warpdrive/
sh autogen.sh  
./configure 
make clean  && make 
make install
### 安装kaezlib
上面准备工作做好以后开始安装
cd /root
cd KAEzip-1.3.11
sh setup.sh install
### 覆盖系统zlib库
cd /usr/local/kaezip/lib
cp  /lib64/libz.so.1.2.11 /opt/        #备份系统zlib库，拷贝到其他地方
cp libz.so.1.2.11  /lib64/             #用kae的zlib库覆盖系统zlib库
mv /lib64/libz.so.1  /lib64/libz.so.1-bak    #软连接改名字，同时备份
ln -s /lib64/libz.so.1.2.11 /lib64/libz.so.1 #覆盖生成新的软连接，这样系统中zlib就被替换成kae zlib库
### 升级pip
 usr/bin/python2 -m pip install --upgrade pip

## 解决python依赖
#第一种命令行安装
pip2 install psutil  pbr lockfile pycparser cffi six bcrypt PyNaCL  ipaddress  enum34  cryptography  paramiko   epydoc   -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com

#第二种脚本安装
#!/bin/bash

list="""
psutil  pbr lockfile pycparser cffi six bcrypt PyNaCL  ipaddress  enum34  cryptography  paramiko   epydoc
"""
for i in $list
do
        echo $i
        pip2 install $i -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com

done

## 插件包安装

### zstd 
cd /root
wget https://github.com/facebook/zstd/archive/refs/tags/v1.4.3.zip 
unzip v1.4.3.zip  
cd zstd-1.4.3/   && make 
### re2c
cd /root
wget https://github.com/skvadrik/re2c/archive/refs/tags/2.0.3.zip
unzip 2.0.3.zip 
cd re2c-2.0.3/  &&./autogen.sh  && ./configure && make  && make install  
### xerces
cd /root
wget https://github.com/greenplum-db/gp-xerces/archive/refs/tags/v3.1.2-p1.zip
unzip v3.1.2-p1.zip
cd /root/gp-xerces-3.1.2-p1/    && ./configure  && make && make install
### ninja
cd /root
wget https://github.com/ninja-build/ninja/archive/refs/tags/v1.10.1.zip 
unzip v1.10.1.zip 
cd ninja-1.10.1/      &&  ./configure.py --bootstrap 
cp  ninja /usr/bin/
### gporca
cd /root
wget https://github.com/greenplum-db/gporca/archive/refs/tags/v3.65.3.zip 
unzip v3.65.3.zip
cd gporca-3.65.3/ && cmake -GNinja -H. -Bbuild
vi libgpos/src/common/CStackDescriptor.cpp
第167行注释掉
[图片]
ninja install -C build
echo  /usr/local/lib >> /etc/ld.so.conf
ldconfig

## 源码安装 greenplum
cd /root
wget https://github.com/greenplum-db/gpdb/archive/refs/tags/6.0.0.zip  
unzip 6.0.0.zip  
cd gpdb-6.0.0/  
./configure --with-perl --with-python --with-libxml --prefix=/usr/local/gpdb  
如果遇到报错zstd 已经安装 ，请执行 --without-zstd，那么可以按照提示加上--without-zstd再次安装
make && make install 
## 检测调用库文件
[图片]
## 集群安装
groupadd -g 3030 gpadmin
useradd -u 3030 gpadmin -g gpadmin -d /home/gpadmin
passwd gpadmin
{输入gpadmin用户新密码}.  ##密码自己设置
## hosts解析
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 hw-gpdb-cluster
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
175.40.42.83 Malluma
127.0.0.1    hw-gpdb-cluster  #解析主机名
175.40.42.83 hw-gpdb-cluster  #解析主机名
## ssh 免密钥
```
su - gpadmin    (切换到gpadmin用户)
mkdir ~/.ssh    (当前模块的以下步骤均在gpadmin用户下执行)
cd ~/.ssh
 ssh-keygen -t rsa     (根据提示按回车，一直到生成秘钥的随机图像完成)
ssh hw-gpdb-cluster   (本地hosts先做解析，然后输入yes，输入密码)  
cd .ssh && cat /home/gpadmin/.ssh/id_rsa.pub >>authorized_keys (上面命令成功后，拷贝密钥)
chmod 600 ~/.ssh/authorized_keys 
ssh hw-gpdb-cluster date (测试免密钥是否成功，输出date命令时间，代表成功)
```
## 建立数据目录
```
mkdir -pv /data/greenplum/master
mkdir -pv /data/greenplum/gp1
mkdir -pv /data/greenplum/gp2
```
## 修改目录权限
```
chown -R gpadmin:gpadmin /usr/local/gpdb/*
chown -R gpadmin:gpadmin /data/greenplum/master
chown -R gpadmin:gpadmin /data/greenplum/gp1
chown -R gpadmin:gpadmin /data/greenplum/gp2
chown gpadmin:gpadmin /usr/local/gpdb 
```
## 修改环境变量
```
su - gpadmin
vi ~/.bash_profile
#添加下面俩行
source /usr/local/gpdb/greenplum_path.sh
export MASTER_DATA_DIRECTORY=/home/greenplum/master/gpseg-1
#加载环境变量
source ~/.bash_profile
```
## 初始化master主机
```
在~目录下增加一个all_hosts_file文件，记录greenplum集群的所有主机
因为搭建的是单台主机，所以集群中只有hw-gpdb-cluster一台主机
echo "hw-gpdb-cluster" >> ~/all_hosts_file
```
## 验证用户免密钥是否有效
```
gpssh-exkeys -f ~/all_hosts_file
```
## 初始化Greenplum数据库系统
```
#拷贝master配置文件
cp /usr/local/gpdb/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin/
#修改配置文件，添加如下配置
declare -a DATA_DIRECTORY=(/data/greenplum/gp1  /data/greenplum/gp2 )
MASTER_HOSTNAME=hw-gpdb-cluster  # MASTER_HOSTNAME主实例的主机名
MASTER_DIRECTORY=/data/greenplum/master    # 主实例的目录
DATABASE_NAME=hwgpadmin    # DATABASE_NAME初始数据库的数据库名
```
## 增加Seg节点

因为搭建的是单台主机，所以集群中段实例也只有hw-gpdb-cluster一台主机。
```
#添加主机
 echo "hw-gpdb-cluster" >> ~/seg_hosts_file
#初始化Seg节点
gpinitsystem -c ~/gpinitsystem_config -h ~/seg_hosts_file
```
## 连接数据库
```
psql -d hwgpadmin
#登陆成功如下： 
hwgpadmin=# select * from gp_segment_configuration;
 dbid | content | role | preferred_role | mode | status | port |    hostname     |     address     |            datadir
------+---------+------+----------------+------+--------+------+-----------------+-----------------+--------------------------------
    1 |      -1 | p    | p              | n    | u      | 5432 | hw-gpdb-cluster | hw-gpdb-cluster | /home/greenplum/master/gpseg-1
    2 |       0 | p    | p              | n    | u      | 6000 | hw-gpdb-cluster | hw-gpdb-cluster | /home/greenplum/gp1/gpseg0
    3 |       1 | p    | p              | n    | u      | 6001 | hw-gpdb-cluster | hw-gpdb-cluster | /home/greenplum/gp2/gpseg1
(3 rows)
状态查看
```

## greenplum数据库操作 
常用命令
### 启动
gpstart

### 停止
gpstop
gpstop -M fast

### 重启
```
gpstop -r# 重新加载配置文件
gpstop -u# 以维护模式启动master
#只启动Master来执行维护或者管理任务而不影响Segment上的数据。
#例如，可以用维护模式连接到一个只在Master实例上的数据库并且编辑系统目录设置。 更多有关系统目录表的信息请见Greenplum Database Reference Guide
gpstart -m #维护模式打开数据库
PGOPTIONS=’-c gp_session_role=utility’ psql postgres #以维护模式连接到master
gpstop -mr #停止维护模式,以正常模式重启数据库# 查看gp配置参数
psql -c 'show all'
#或者
gpconfig -s <参数名>
```

### 修改gp参数
```
#设置max_connections,在master上设置为10, 在segment上设置为100
#-v指定所有节点,包括master,standby,segment
#-m指定master节点
#–master-only指定只修改master节点
#-r移除参数的配置
#-l显示所有可配置的参数
gpconfig -c max_connections -v 100 -m 10
```

### 查看gp状态
```
gpstate #查看简要信息
gpstate -s #查看详细信息
gpstate -m #查看镜像配置
gpstate -c #查看镜像的映射关系
gpstate -f #查看standby状态# 恢复
gprecoverseg # 恢复失败的 segment 实例
gprecoverseg -F -v # 全量恢复
gprecoverseg -r # 还原所有Segment的角色
```

### 关闭集群
gpstop -a -M fast

以restricted方式启动数据库
gpstart -a -R 

### 开始修复故障节点
gprecoverseg -a

### 查看修复状态 
gpstate -m

### 重启greenplum集群
```
gpstop -a -r
sql 语句
#查看所有segment节点信息
select * from gp_segment_configuration;
#查看节点主机磁盘空闲空间
SELECT * FROM gp_toolkit.gp_disk_free ORDER BY dfsegment;
#查看数据库的使用空间
SELECT * FROM gp_toolkit.gp_size_of_database ORDER BY sodddatname;
#查看表的磁盘使用空间
SELECT relname AS name, sotdsize AS size, sotdtoastsize 
   AS toast, sotdadditionalsize AS other 
   FROM gp_toolkit.gp_size_of_table_disk as sotd, pg_class 
   WHERE sotd.sotdoid=pg_class.oid ORDER BY relname;
#查看索引的使用空间
SELECT soisize, relname as indexname
    FROM pg_class,gp_toolkit.gp_size_of_index
    WHERE pg_class.oid=gp_size_of_index.soioid 
    AND pg_class.relkind='i';
#查看表的分布键
\d+ table_name
#查看表的数据分布
SELECT gp_segment_id, count(*) 
   FROM table_name GROUP BY gp_segment_id;
#查看当前正在运行的会话
select pid,usename,datname,waiting_reason,query,client_addr from pg_stat_activity where state='active';
#查看当前等待的会话
select pid,usename,datname,query,client_addr,waiting_reason from pg_stat_activity where waiting ;
#杀死会话
select pg_cancel_backedn(111);select pg_terminate_backend(pid) from pg_stat_activity where ;   -- 必须要接条件,而且最好先查出你需要的杀的会话.
select 'select pg_terminate_backend('||pid||');' from pg_stat_activity where    ;   --必须要接条件
select 'select pg_cancel_backend('||pid||');' from pg_stat_activity where    ;   --

#-- 查看资源队列的配置
select * from pg_resqueue_attributes;
#-- 查看资源队列的使用情况
select * from pg_resqueue_status;
#在psql中使用\h command可以获取具体命令的语法
\h create view
```
## 目录结构介绍
[图片]
```
base是数据目录，每个数据库在这个目录下，会有一个对应的文件夹。
global是每一个数据库公用的数据目录。
gpperfmon监控数据库性能时，存放监控数据的地方。
pg_changetracking是Segment之间主备同步用到的一些原数据信息保存的地方。
pg_clog是记录数据库事务信息的地方，保存了每一个事务id的状态，这个非常重要，不能丢失，一旦丢失，整个数据库就基本上不可用了。
pg_log是数据库的日志信息。
pg_twophase是二阶段提交的事务信息（关于二阶段提交的内容可参阅第7章中的介绍）
pg_xlog是数据库重写日志保存的地方，其中每个文件固定大小为64MB，并不断重复使用。
gp_dbid记录这个数据库的dbid以及它对应的mirror节点的dbid。
```
