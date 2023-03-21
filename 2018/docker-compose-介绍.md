### pip 安装

#### pip 更改为国内资源下载
```
[root@c7-vm2_32 ~]# cat >> ~/.pip/pip.conf << "EOF"
[global]
index-url=https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
EOF
```
#### epel 源下载
```
rpm -ivh http://mirrors.aliyun.com/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
```
#### 安装pip
```
yum -y install certbot libevent-devel gcc libffi-devel python-devel openssl-devel python2-pip
```
#### 升级pip版本

安装以后版本是
```
[root@c7-vm2_32 ~]# pip --version
pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)
```
pip 升级版本
```
[root@c7-vm2_32 ~]# pip install --upgrade pip
[root@c7-vm2_32 ~]# pip --version
pip 9.0.1 from /usr/lib/python2.7/site-packages (python 2.7)
```
#### 安装docker-compose

```
pip install -U docker-compose
#pypi源比较卡，执行下面命令可以使用豆瓣源来安装
pip install -U docker-compose  -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
```
```
[root@c7-vm2_32 ~]# docker-compose --version
docker-compose version 1.17.1, build 6d101fb
[root@c7-vm2_32 ~]# which docker-compose
/bin/docker-compose
```

### curl 方式安装docker-compose
#### curl 命令
```
curl -L https://github.com/docker/compose/releases/download/1.14.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose -version
```
### 总结
curl方式安装docker-compose 比较慢，
强烈建议使用pip方式安装docker-compose。
