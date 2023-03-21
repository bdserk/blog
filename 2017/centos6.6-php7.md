##  编译php7
<!-- more -->
### 机器环境
```
CentOS release 6.6 (Final)
kernal 2.6.32-504.23.4.el6.x86_64

```

### yum

yum install -y  curl libcurl-devel libjpeg-devel libpng-devel libjped-devel freetype-devel libxslt-devel boost-devel gperf libevent-devel libuuid-devel libgearman  libgearman-devel

### install php
下载目录: `/opt/`
安装目录: `/Data/apps/php/`

```
cd /opt
wget http://docs.php.net/distributions/php-7.0.28.tar.gz
tar xf php-7.0.28.tar.gz
cd php-7.0.28
./configure --with-libdir=lib64 --prefix=/Data/apps/php --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-gd --with-zlib --with-png-dir --with-jpeg-dir --with-iconv --with-curl --with-mcrypt --with-openssl --with-xsl --enable-opcache --enable-inline-optimization --enable-fpm --enable-mbstring --enable-pcntl --enable-soap --enable-sockets --enable-bcmath --with-libxml --with-freetype-dir=/usr/include/freetype2/ --disable-phar

make && make install
```

### php 扩展
> 下面所有php扩展包的下载目录统一为: `/opt/soft/`

#### opcache

```
cd /opt/php-7.0.28/ext
/Data/apps/php/bin/phpize
./configure --with-php-config=/Data/apps/php/bin/php-config
make && make install
```

#### xdebug
`从2.4开始支持php7`

下载地址: <https://xdebug.org/files/>

```
wget https://xdebug.org/files/xdebug-2.6.0.tgz
tar xf xdebug-2.6.0.tgz
cd xdebug-2.6.0
/Data/apps/php/bin/phpize
./configure --with-php-config=/Data/apps/php/bin/php-config
make
make install
```

#### igbinary
`最新版本(2.0.5),2.0.1开始支持7.0`

详情连接: <http://pecl.php.net/package-changelog.php?package=igbinary>

```
wget https://pecl.php.net/get/igbinary-2.0.5.tgz
tar xf igbinary-2.0.5.tgz
cd igbinary-2.0.5
/Data/apps/php/bin/phpize
./configure --with-php-config=/Data/apps/php/bin/php-config
make
make install
```

#### memcached

```
memcached版本要求：
	php-memcached 3.x
	Supports PHP 7.0 - 7.2.
	Requires libmemcached 1.x or higher.
	Optionally supports igbinary 2.0 or higher.
	Optionally supports msgpack 2.0 or higher.
```

```
安装libmemcached 依赖包
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gzc
tar -zxf libmemcached-1.0.18.tar.gz
./configure --prefix=/Data/apps/libmemcached --with-memcached
make && make install
```

```
wget https://pecl.php.net/get/memcached-3.0.0.tgz
tar xf memcached-3.0.0.tgz
cd /opt/soft/memcached-3.0.0
/Data/apps/php/bin/phpize
./configure --with-php-config=/Data/apps/php/bin/php-config --with-libmemcached-dir=/Data/apps/libmemcached --enable-memcached --enable-memcached-igbinary
make && make install
```

遇到这样的报错
`configure: error: no, sasl.h is not available. Run configure with --disable-memcached-sasl to disable this check`
根据提示加上参数重新编译

```
./configure --with-php-config=/Data/apps/php/bin/php-config --with-libmemcached-dir=/Data/apps/libmemcached --enable-memcached --enable-memcached-igbinary  --disable-memcached-sasl
make && make install
```
#### imagick

下载地址：`https://pecl.php.net/package/imagick`

```
安装ImageMagick(ImageMagick-7.0.7-28)
wget ftp://mirror.checkdomain.de/imagemagick/ImageMagick-7.0.7-28.tar.gz
tar xf ImageMagick-7.0.7-28.tar.gz
cd ImageMagick-7.0.7-28
./configure --prefix=/Data/apps/ImageMagick
make && make install

```

```
编译imagick
wget http://pecl.php.net/get/imagick-3.4.3.tgz
tar xf imagick-3.4.3.tgz
cd imagick-3.4.3
/Data/apps/php/bin/phpize
./configure --with-imagick=/Data/apps/ImageMagick --with-php-config=/Data/apps/php/bin/php-config
make && make install

```

#### redis

下载地址: `https://github.com/phpredis/phpredis  (develop版本)`

安装redis目录: `/Data/app/redis`

```
编译redis扩展
wget https://pecl.php.net/get/redis-3.0.0.tgz
cd redis-3.0.0
/Data/apps/php/bin/phpize
./configure --enable-redis-igbinary=/Data/apps/redis/bin/ --with-php-config=/Data/apps/php/bin/php-config
make && make install
```
#### gearman

安装gearmand服务端 <https://launchpad.net/gearmand> 版本：1.1.12

```
编译gearman客服端扩展
git下载最新：https://github.com/wcgallego/pecl-gearman/tree/master
cd pecl-gearman-master
/Data/apps/php/bin/phpize
./configure  --with-php-config=/Data/apps/php/bin/php-config
make && make install
```
#### scws

下载链接：<http://www.xunsearch.com/scws/download.php>

```
wget http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2
cd scws-1.2.3/phpext
/Data/apps/php/bin/phpize
./configure --with-scws=/Data/apps/scws --with-php-config=/Data/apps/php/bin/php-config
```

#### amqp

下载地址: <http://pecl.php.net/package/amqp>

```
安装rabbitmq-c依赖库
wget https://github.com/alanxz/rabbitmq-c/releases/download/v0.8.0/rabbitmq-c-0.8.0.tar.gz
cd rabbitmq-c-0.8.0
./configure --prefix=/usr/local/rabbitmq-c-0.8.0
make && make install

```

```
编译amqp扩展
wget https://pecl.php.net/get/amqp-1.9.3.tgz
tar -xf amqp-1.9.3.tar
cd amqp-1.9.3
/Data/apps/php/bin/phpize
./configure --with-php-config=/Data/apps/php/bin/php-config --with-amqp --with-librabbitmq-dir=/usr/local/rabbitmq-c-0.8.0
make && make install

```

#### fastdfs client

```
wget https://github.com/happyfish100/fastdfs/archive/master.zip
unzip master.zip
cd fastdfs-master/php_client
/Data/apps/php/bin/phpize
./configure --with-php-config=/Data/apps/php/bin/php-config
make && make install
```
####  libiconv

```
安装libiconv
wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
tar xf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure
make
make install
```

```
安装libdatrie
解压，进入目录
./configure LDFLAGS=-L/usr/local/lib LIBS=-liconv --host=arm
make
make install
```

```
安装trie_filter.so 拓展
git clonde https://github.com/zzjin/php-ext-trie-filter
cd  php-ext-trie-filter
/Data/apps/php/bin/phpize
./configure  --with-php-config=/Data/apps/php/bin/php-config  --with-trie_filter=/usr/local/libdatrie
make && make install
```

### php.ini 配置

```
cd /opt/soft/php-7.0.28
cp php.ini-production /Data/apps/php/lib/php.ini
```

```
加载的模块配置
[opcache]
zend_extension="/Data/apps/php/lib/php/extensions/no-debug-non-zts-20151012/opcache.so"
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.fast_shutdown=1
opcache.enable_cli=1
opcache.validate_timestamps=1
opcache.revalidate_freq=1
opcache.error_log="/Data/apps/php/var/log/opcache.log"
[memcached]
extension=memcached.so
memcache.hash_strategy=consistent
memcache.hash_function=crc32
session.save_handler = memcached
extension=igbinary.so
extension=imagick.so
extension=redis.so
extension=gearman.so
extension=trie_filter.so
[scws]
extension=scws.so
scws.default.charset = utf8
scws.default.fpath = /Data/apps/scws/etc
[amqp]
extension=amqp.so
[fastdfs]
extension = fastdfs_client.so
fastdfs_client.base_path = /tmp
fastdfs_client.connect_timeout = 2
fastdfs_client.network_timeout = 60
fastdfs_client.log_level = info
fastdfs_client.http.anti_steal_secret_key =
fastdfs_client.tracker_group_count = 1
fastdfs_client.tracker_group0 = /etc/fdfs/client.conf
fastdfs_client.use_connection_pool = false
fastdfs_client.connection_pool_max_idle_time = 3600

```
### php-fpm.conf

```
cp /Data/apps/php/etc/php-fpm.conf.default /Data/apps/php/etc/php-fpm
```

### php-fpm 启动脚本

```
cd /opt/soft/php-7.0.28/sapi/fpm
cp init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
```

### 启动php

```
/etc/init.d/php-fpm start
```
