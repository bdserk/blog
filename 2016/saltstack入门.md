---
![img](http://upload-images.jianshu.io/upload_images/1542757-08495ea71b239951.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## saltstack介绍
   关于Saltstack的介绍，简单一句话：整合了Puppet和 Chef的功能，更加强大，更适合大规模批量管理服务器。
* 开发语言:  **python** 
* 工作方式:  **master/minion(zeroMQ) ,masterless  , salt-ssh(0.17+)**
* 三大方式:   
       remote exection:  远程执行
       config mamagement: 配置管理 
       cloud mangement(?): 云端的管理

## 基本概念
Saltstack基于C/S架构，服务端master和客户端minions
* 部署简单、方便；
* 支持大部分UNIX/Linux及Windows环境；
* 主从集中化管理；
*  配置简单、功能强大、扩展性强；
* 主控端（master）和被控端（minion）基于证书认证，安全可靠；
* 支持API及自定义模块，可通过Python轻松扩展         

>Master : 控制中心,salt命令运行和资源状态管理
Minions : 需要管理的客户端机器,会主动去连接Mater端,并从Master端得到资源状态
信息,同步资源管理信息
States: 配置管理的指令集
Modules: 在命令行中和配置文件中使用的指令模块,可以在命令行中运行
Grains: minion端的变量,静态的
Pillar:minion端的变量,动态的比较私密的变量,可以通过配置文件实现同步minions定义
highstate: 为minion端下发永久添加状态,从sls配置文件读取.即同步状态配置
salt_schedule: 会自动保持客户端配置




## Master与Minion认证 

* minion在第一次启动时，会在/etc/salt/pki/minion/（该路径在/etc/salt/minion里面设置）下自动生成minion.pem（private key）和 minion.pub（public key），然后将 minion.pub发送给master。
* master在接收到minion的public key后，通过salt-key命令accept minion public key，这样在master的/etc/salt/pki/master/minions下的将会存放以minion id命名的 public key，然后master就能对minion发送指令了。
 
## master与minion之间的通信
* SaltStack master启动后默认监听4505和4506两个端口。
* 4505（publish_port）为saltstack的消息发布系统
* 4506（ret_port）为saltstack客户端与服务端通信的端口，负责接收客户端发送过来的结果
---
- ![img](http://upload-images.jianshu.io/upload_images/1542757-0749dd9d601d40c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##安装+配置
**配置域名hosts文件,也可以使用ip地址**
~~~python
cat /etc/hosts
10.10.10.14 test14.salt.cn 
10.10.10.15 master15.salt.cn
~~~

## 环境系统
~~~python
master: 10.10.10.15 
minion: 10.10.10.14
~~~

## yum安装

 **master端:**
 ~~~python
 yum install salt-master
 ~~~
**配置文件地址: /etc/salt/master** 
 _启动的端口: _
 * 4505(publisth_port) : 用于salt的消息发布系统
 * 4506(retrun_port):  salt客户端与服务器端通信的端口,接受客户端返回结果

** 启动服务: **
~~~python
/etc/init.d/salt-master 
~~~
**minion端:**
~~~python 
yum install  salt-minion
~~~

**配置文件地址: /etc/salt/minion**
~~~python
master: 10.10.10.15　                   #服务器端的主机名
id:  test14.salt..cn                          #客户端的主机名 ,唯一标识
~~~
*注意: 配置文件要格式统一 *
**启动服务:**
~~~python 
/etc/init.d/salt-minion start 
~~~

## 通信认证
salt 的master和minion 之间是通过证书通信的. 所以存在证书的信任颁发问题
** master端**  
~~~python
salt-key -L  : 查看当前需要接受的keys 
salt-key-a hostname   : 接受来自一个主机的证书
salt-key-A  ：接受所有请求的证书 
/etc/salt/master 配置文件 : 自动接受请求证书 
    auto_accept: true 
~~~
~~~python
[root@bj-idc-15 master]# salt-key -L   
Accepted Keys:
Denied Keys:
Unaccepted Keys:
test14.salt.cn                             #未被认证的主机
Rejected Keys:
~~~
~~~python
[root@bj-idc-15 master]# salt-key -a test14.salt.cn    
The following keys are going to be accepted:
Unaccepted Keys:
test14.salt.cn
Proceed? [n/Y] y
Key for minion test14.salt.cn accepted.      #请求认证已经被接受 
~~~

## 测试通信 
~~~python
[root@bj-idc-15 master]# salt "test14.salt.cn"  test.ping 
test14.salt.cn:
    True 
~~~
*返回True代表成功了* 
~~~python
[root@bj-idc-15 master]# mkdir -p /srv/salt ; cd /srv/salt
[root@bj-idc-15 salt]# mkdir /srv/salt/dev 
[root@bj-idc-15 salt]# cat top.sls 
base:
  '*': 
    - dev.http 
~~~
~~~python
[root@bj-idc-15 salt]# cat dev/http.sls 
http: 
  pkg:
  - name: httpd 
  - installed 
  service: 
  - name: httpd 
  - running 
  - reload: True 
  - watch: 
    - file: /etc/httpd/conf/httpd.conf 
/etc/httpd/conf/httpd.conf: 
  file.managede:
  - source: salt://files/httpd.conf 
  - user: root 
  - group: root
  - mode: 644 
  - backup: minion 
~~~

## 执行命令
> sls文件写好以后，在master执行命令 

~~~python
[root@bj-idc-15 ~]# salt “test14.salt.cn” state.highstate
~~~

## 检查执行结果
~~~python
[root@bj-idc-14 ~]# rpm -qa httpd
httpd-2.2.15-47.el6.centos.1.x86_64
[root@bj-idc-14 ~]# ss -tunlp | grep 1000 
tcp    LISTEN     0      128                   :::1000                 :::*      users:(("httpd",29395,4),("httpd",29397,4),("httpd",29398,4),("httpd",29399,4),("httpd",29400
~~~
