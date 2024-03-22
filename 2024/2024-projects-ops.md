# 官网服务站点配置

## 前言
公司官网和游戏官网web站点配置
- 系统环境： Linux、MySQL、Nginx、PHP、Redis
- 机器系统： centos 8 或者 opencloud 8
- 系统盘： 建议40G或者50G

## 代码说明
> 后台使用HushFramework框架开发， https://github.com/jameschz/hush，更多配置参考以上开源项目

- 代码目录： /data/code
- 公司官网： fy-site-company  -> 对应正式域名： www.suhemaoan.com 
- 公司后台： fy-site-gmtool   -> 对应正式域名： gmtool.suhemaoan.com
- 游戏官网： fy-site-official -> 对应正式域名： yyfy.suhemaoan.com

## 项目配置
我们以fy-site-company为例子
```
cd /data/code/fy-site-company
ls -la  #可以看到.env.php
```
- `.env.php`: 配置服务环境，基础域名地址
- `global.appcfg.php`: 配置session存储，redis缓存，静态文件服务器，cdn域名地址等
- `database.mysql.php`: 数据库配置，这里是深度的分库分表策略，一般由程序控制
  







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

###  golang env
```
yum install -y golang
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
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
### 数据库初始化
```
php /data/code/fy-site-gmtool/bin/hush.php db init create
php /data/code/fy-site-gmtool/bin/hush.php db table create
php /data/code/fy-site-gmtool/bin/hush.php db table data
php /data/code/fy-site-gmtool/bin/hush.php db userdb
php /data/code/fy-site-gmtool/bin/hush.php db usertable
```
### 编译go代码
`安装和启动fy-server服务`
```
cd /data/code/fy-server
./build.sh
./server.sh
```
