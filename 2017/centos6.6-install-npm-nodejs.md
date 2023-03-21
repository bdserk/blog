### redis
```
#### 安装
yum install redis
###设置密码
#### 配置文件中
requirepass  123456
#### 命令行模式
config set requirepass 123456
#### 连接测试
redis-cli -p 6379 -a 123456
```
#### rabbitmq
```
#### 安装
yum install rabbitmq
#### 新建用户和密码
rabbitmqctl  add_user bds bds123 
#### 设置管理员
rabbitmqctl  set_user_tags yao administrator
#### 设置权限
rabbitmqctl  set_permissions -p / bds ".*" ".*" ".*"

```
### mysql
``` 
#### 安装
yum install mysql 
#### 创建数据库
"create database zabbix default character set utf8 collate utf8_bin;"
#### 创建权限
grant all on zabbix.* to 'zabbix'@localhost identified by 'zabbix';
```
### git 1.9
```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
cd /opt
wget https://www.kernel.org/pub/software/scm/git/git-1.9.4.tar.gz
tar xzf git-1.9.4.tar.gz
cd git-1.9.4
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc
```
### nodejs和npm YUM安装
```
[root@c6-23 ~]# curl --silent --location https://rpm.nodesource.com/setup_9.x | bash -
[root@c6-23 ~]# yum install nodejs -y
[root@c6-23 ~]# node -v
v9.11.2
[root@c6-23 ~]# npm -v
5.6.0
### npm 升级
[root@c6-23 ~]# npm i -g npm
/usr/bin/npm -> /usr/lib/node_modules/npm/bin/npm-cli.js
/usr/bin/npx -> /usr/lib/node_modules/npm/bin/npx-cli.js
+ npm@6.8.0
added 314 packages, removed 364 packages and updated 52 packages in 22.254s
```
### nodejs和npm 二进制安装
```
wget  https://nodejs.org/dist/v11.10.0/node-v11.10.0-linux-x64.tar.xz
tar xvJf  node-v11.10.0-linux-x64.tar.xz
cp node-v11.10.0-linux-x64 /Data/apps/node
ln -sv  /Data/apps/node/bin/node  /usr/sbin/node
ln -sv  /Data/apps/node/bin/npm   /usr/sbin/npm 
```
### 更改淘宝镜像
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
### vue 
```
npm install vue -g
```
