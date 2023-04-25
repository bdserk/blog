## yum安装
```
yum -y install ncurses ncurses-devel libaio-devel gmp gmp-devel  mpfr  mpfr-devel  libmpc libmpc-devel zlib-devel net-tools cmake openssl openssl-devel gcc-c++  rpcgen libtirpc libtirpc-devel  libstdc++-static autoconf automake libtool
```
## 编译cmake
```
cd /root
wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz --no-check-certificate
tar xf cmake-3.5.2.tar.gz
cd  cmake-3.5.2
 ./bootstrap
make -j 96   && make install  && hash -r 
/usr/local/bin/cmake --version
```
## 编译gcc
```
cd /root/
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz --no-check-certificate
tar xf  gcc-7.3.0.tar.gz 
cd gcc-7.3.0
./configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --enable-bootstrap
make -j 96 && make install 
gcc -v
```
## 编译安装kaezip
### 下载kaezip
```
cd /root
wget https://github.com/kunpengcompute/KAEzip/archive/refs/tags/v1.3.11.tar.gz
 tar xf v1.3.11.tar.gz
cd KAEzip-1.3.11/
```
### 下载zlib
```
wget https://nchc.dl.sourceforge.net/project/libpng/zlib/1.2.11/zlib-1.2.11.tar.gz

#把zlib拷贝到kae zip 相应的目录中

cp  zlib-1.2.11.tar.gz KAEzip-1.3.11/open_source/     
```
### 下载KAEdriver
```
cd /opt/
wget https://github.com/kunpengcompute/KAEdriver/archive/refs/tags/v1.3.11.tar.gz
tar xf v1.3.11.tar.gz
cd KAEdriver-1.3.11/warpdrive/
sh autogen.sh  
./configure 
make clean  && make 
make install
```
### 安装kaezlib
上面准备工作做好以后开始安装
```
cd /root
cd KAEzip-1.3.11
sh setup.sh install
```
### 覆盖系统zlib库
```
cd /usr/local/c/lib
cp  /lib64/libz.so.1.2.11 /opt/        #备份系统zlib库，拷贝到其他地方
cp libz.so.1.2.11  /lib64/             #用kae的zlib库覆盖系统zlib库
mv /lib64/libz.so.1  /lib64/libz.so.1-bak    #软连接改名字，同时备份
ln -s /lib64/libz.so.1.2.11 /lib64/libz.so.1 #覆盖生成新的软连接，这样系统中zlib就被替换成kae zlib库
```
## 编译安装MySQL
### 下载MySQL
```
cd /root
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.27.tar.gz --no-check-certificate
tar xf mysql-boost-5.7.27.tar.gz
cd mysql-boost-5.7.27.tar.gz
```
### 解决依赖
```
wget https://fr-repo.openeuler.org/openEuler-20.03-LTS-SP3/update/aarch64/Packages/libtirpc-devel-1.2.6-2.oe1.aarch64.rpm
rpm -ivh libtirpc-devel-1.2.6-2.oe1.aarch64.rpm
wget https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4.1/rpcsvc-proto-1.4.1.tar.xz
xz -d rpcsvc-proto-1.4.1.tar.xz
tar -xvf rpcsvc-proto-1.4.1.tar
cd rpcsvc-proto-1.4.1
./configure
make && make install
```
### 开始编译MySQL
```
cd /root
cd mysql-5.7.27/
mkdir build  && cd build 
cmake .. -DBUILD_CONF\FIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/root/mysql-5.7.27/boost/boost_1_59_0   -DWITH_ZLIB=system       
make -j 96
make install  
/usr/local/mysql/bin/mysql --version
```
### 新建用户和目录授权
```
useradd mysql 
mkdir -pv /data/mysql/data 
cd /data   && chown mysql.mysql  -R  mysql/
```
### 添加环境变量
```
echo "export PATH=$PATH:/usr/local/mysql/bin" >> /etc/profile.d/mysql.sh 
source  /etc/profile.d/mysql.sh 
```
### 查看MySQL调用库
```
ldd /usr/local/mysql/bin/mysqld
[图片]
https://bugs.mysql.com/bug.php?id=97547
```
