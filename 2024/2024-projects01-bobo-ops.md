# 官网服务站点配置

## 前言
公司官网和游戏官网web站点配置
- 系统环境： MySQL8.0+、Nginx、PHP、Redis5.0+
- 机器系统： centos 8 或者 opencloud 8
- 系统盘： 建议40G或者50G

## 代码说明
> 后台使用HushFramework框架开发 , <https://github.com/jameschz/hush> , 更多配置参考以上开源项目

- 公司官网： fy-site-company  -> 对应正式域名： www.suhemaoan.com 
- 公司后台： fy-site-gmtool   -> 对应正式域名： gmtool.suhemaoan.com
- 游戏官网： fy-site-official -> 对应正式域名： yyfy.suhemaoan.com
- 基础代码： fy-site-base
- 代码目录： /data/code

## 基础环境安装
### system 
```
yum install -y epel-release
yum install -y redis nginx
yum install -y mysql mysql-common mysql-server
```
### php env
```
yum install -y php php-cli php-fpm php-common php-devel php-embedded php-gd php-mbstring php-mysqlnd php-opcache php-pdo php-xml php-bcmath php-zip php-json php-redis
yum install -y php-pear
pecl install redis
echo 'extension=redis' > /etc/php.d/50-php_redis.ini
php -m
```
### 域名配置文件
```
\cp -f ./fy-gmtool.conf /etc/nginx/conf.d/
```
### 启动服务命令
```
systemctl enable nginx; systemctl start nginx
systemctl enable redis; systemctl start redis
systemctl enable mysqld; systemctl start mysqld
systemctl enable php-fpm; systemctl start php-fpm
```
### 设置数据登录
```
mysql -u root -p
USE mysql;
#创建用户
CREATE USER 'root'@'%' IDENTIFIED BY '新密码';
# 授权远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'  WITH GRANT OPTION;
或者
# 设置root密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
# 授权远程主机登录（将%替换为特定的远程主机IP，如果要授权所有主机，则使用%
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '新密码' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 数据库初始化
```
php /data/code/fy-site-gmtool/bin/hush.php db init create
php /data/code/fy-site-gmtool/bin/hush.php db table create
php /data/code/fy-site-gmtool/bin/hush.php db table data
php /data/code/fy-site-gmtool/bin/hush.php db userdb
php /data/code/fy-site-gmtool/bin/hush.php db usertable
```
## 项目配置

### 基础环境配置
我们以fy-site-company为例子
```
cd /data/code/fy-site-company
ls -la  #可以看到.env.php
```
- `.env.php`: 配置服务环境，基础域名地址
- `global.appcfg.php`: 配置session存储，redis缓存，静态文件服务器，cdn域名地址等
- `database.mysql.php`: 数据库配置，这里是深度的分库分表策略，一般由程序控制

### 数据库配置
由于后台站点会共用数据库，所以具体的数据库配置文件在fy-site-base/lib/App/Db目录（分环境、支持分库分表），控制所有后台站点的数据库

注意：`所有配置文件都可以分环境，比如：.env.php里面的ENV是release则系统会采用etc/release下面的配置文件`

![](https://gitee.com/budongshu/blogimg/raw/master/img/202403222038468.png)



# 游戏服务器
> 后台全部使用GoBase框架开发（https://github.com/jameschz/go-base）

- fy-server 游戏服务器
- fy-service 专门的游戏服务器
  
## 机器环境
游戏服务器配置
- 系统环境： MySQL8.0,Redis5.0+,PHP,Nginx,Golang1.18
- 机器系统： centos 8 或者 opencloud 8
- 系统盘： 建议40G或者50G

## 代码目录
- /data/code/fy-server/ ： 代码目录
- /data/code/fy-service/ ： 代码目录
- /data/code/fy-server/bin： 二进制执行目录
- /data/code/fy-server/server.sh： 执行启动服务脚本
- /data/code/fy-server/etc/： 配置文件目录，同理，不同环境分为不同目录

### system 
```
yum install -y epel-release
yum install -y redis nginx
yum install -y mysql mysql-common mysql-server
```
### php env
```
yum install -y php php-cli php-fpm php-common php-devel php-embedded php-gd php-mbstring php-mysqlnd php-opcache php-pdo php-xml php-bcmath php-zip php-json php-redis
yum install -y php-pear
pecl install redis
echo 'extension=redis' > /etc/php.d/50-php_redis.ini
php -m
```
###  install golang env
```
wget https://golang.google.cn/dl/go1.18.10.linux-amd64.tar.gz
tar xf go1.18.10.linux-amd64.tar.gz  -C /usr/local/
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
echo "export GOROOT=/usr/local/go  >> /etc/profile.d/go.sh 
echo "export PATH=$PATH:$GOROOT/bin:$GOPATH/bin" >> /etc/profile.d/go.sh
source /etc/profile.d/go.sh
```

## 配置目录
我们以线上稳定环境版本为例子
- /data/code/fy-server/etc/env.txt: 配置服务环境
- /data/code/fy-server/etc/release/config.yaml : 服务启动地址和端口
- /data/code/fy-server/etc/release/database.yaml: 配置数据库等地址，支持分库，分表
- /data/code/fy-server/etc/release/cache.yaml: 配置redis等缓存数据库地址和集群

### 编译go代码
- 第一种：直接通过镜像系统机器或者从其他机器拷贝过来源代码以及服务启动二进制文件，然后通过启动命令，进行服务启动
- 第二种：通过make 编译，重新生成服务启动二进制文件，然后通过启动命令，进行服务启动

**目前是采用第一种方式启动服务** 

`安装和启动fy-server服务`
```
cd /data/code/fy-server
./build.sh
./server.sh
```
