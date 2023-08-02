
### 安装kae zlib库
 ```
 wget https://github.com/kunpengcompute/KAE/releases/download/v1.3.11/kae_1.3.11_openEuler20.03LTS.zip
 tar xf kae_1.3.11_openEuler20.03LTS.zip
cd kae_1.3.11_openEuler20.03LTS
rpm -ivh uacce*.rpm hisi*.rpm libwd-*.rpm libkae*.rpm
rpm -qa|grep -E "hisi|uacce|libwd|libkae"
```
### yum安装
```
yum -y install ncurses ncurses-devel libaio-devel gmp gmp-devel mpfr mpfr-devel libmpc libmpc-devel zlib-devel net-tools cmake openssl openssl-devel gcc-c++
```
### 编译cmake   

```
cd /root 
wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz --no-check- certificate 
tar xf cmake-3.5.2.tar.gz &&  cd cmake-3.5.2 
./bootstrap 
make -j 96 && make install && hash -r && /usr/local/bin/cmake --version
```
### 解决依赖问题
```
yum install curl-devel -y
pip install Cheetah  typing pyyaml
```
### 下载
```
wget https://github.com/mongodb/mongo/archive/r4.0.12.tar.gz --no-check-certificate
tar xf r4.0.12.tar.gz
cd mongo-r4.0.12/
```
### 编译
```
python2 buildscripts/scons.py MONGO_VERSION=4.0.12 all CFLAGS="-march=armv8-a+crc -mtune=generic" -j 64 --disable-warnings-as-errors -Wunused-result
```
### 安装
```
python2 buildscripts/scons.py MONGO_VERSION=4.0.12 --prefix=/usr/local/mongo --disable-warnings-as-errors CFLAGS="-march=armv8-a+crc" install -j 64
```
清理临时文件
```
cd /usr/local/mongo/bin  
strip mongos && strip mongod && strip mongo 
```
### 启动
```
mkdir /data/mongo/data -pv
/usr/local/mongo/bin/mongod --dbpath=/data/mongo/data --logpath=/usr/local/mongo/logs --logappend --port=27017 --fork
```
### 调用库检测
```
ldd /usr/local/mongo/bin/mongod
```
