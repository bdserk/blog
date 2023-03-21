### 安装jdk
```
yum install -y java-1.7.0-openjdk
```
### 下载
```
cd /root/
wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz
```
### 解压拷贝
```
tar xf zookeeper-3.4.8.tar.gz  -C  /Data/apps/ 
cd /Data/apps/
mv zookeeper-3.4.8  zookeeper 
chown root.root -R zookeeper/ 
cp zookeeper/conf/zoo_sample.cfg  zookeeper/conf/zoo.cfg
```
### 环境变量
```
echo 'export PATH=$PATH:/Data/apps/zookeeper/bin/' >  /etc/profile.d/zookeeper
. /etc/profile.d/zookeeper
```
### 启动
```
/Data/apps/zookeeper/bin/zkServer.sh start 
```
