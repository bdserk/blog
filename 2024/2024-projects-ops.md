


## 基础环境安装

### system 
yum install -y epel-release
yum install -y redis nginx
yum install -y mysql mysql-common mysql-server

### php env
yum install -y php php-cli php-fpm php-common php-devel php-embedded php-gd php-mbstring php-mysqlnd php-opcache php-pdo php-xml php-bcmath php-zip php-json php-redis
yum install -y php-pear
pecl install redis
echo 'extension=redis' > /etc/php.d/50-php_redis.ini
php -m

###  golang env
yum install -y golang
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct


### 域名配置文件
\cp -f ./fy-gmtool.conf /etc/nginx/conf.d/

### 启动服务命令
systemctl enable nginx; systemctl start nginx
systemctl enable redis; systemctl start redis
systemctl enable mysqld; systemctl start mysqld
systemctl enable php-fpm; systemctl start php-fpm

### 数据库初始化
php /data/code/fy-site-gmtool/bin/hush.php db init create
php /data/code/fy-site-gmtool/bin/hush.php db table create
php /data/code/fy-site-gmtool/bin/hush.php db table data
php /data/code/fy-site-gmtool/bin/hush.php db userdb
php /data/code/fy-site-gmtool/bin/hush.php db usertable

### 编译go代码 
`安装和启动fy-server服务`

cd /data/code/fy-server
./build.sh
./server.sh
