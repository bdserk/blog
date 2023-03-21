![image](http://upload-images.jianshu.io/upload_images/1542757-b33ceab34ab338f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## fabric 介绍
* Fabic是一个python(2.5-2.7)库,用于简化使用ssh的应用程序部署,或系统管理任务 
* 它提供的操作包括: 执行本地或者远程shell命令,上传/下载文件,以及其他辅助的功能,如提示用户输入,中止执行等 
Github主页: http://github.com/fabric/fabric

## fabric的安装
> shell 脚本实现 

~~~python
#!/bin/bash 
Install() { 
        yum install python-setuptools python-devel -y
        easy_install  pip
        pip  -v uninstall  pycrypto  
        pip  -v  install   pycrypto  --upgrade 
        pip   install  fabric
} 
Install
[  $? -eq 0 ]  &&  fab -V  || echo "Install fabric error!"
~~~

##常用的fabric接口
- local : 执行本地命令,如local('uname -s')
- lcd:  切换本地目录, *如lcd('/home') ,进入到home目录下*
- cd: 切换远程的目录, *如cd('/tmp')*
- run: 执行远程命令, *如run('free -m')*
- sudo: 以sudo的方式执行远程命令,*如:sudo(/etc/init.d/httpd start)* 
- put: 上传本地文件到远程主机,*put('/etc/fstab','/tmp/fstab.info')*
- get: 从远程主机下载本地文件到主机 ,*get('/etc/fatab','/tmp/fstab.info')*
- prompt:获得用户输入信息交互式,*prompt('plese input user password:')*
- confirm: 获得提示信息确认,*confirm('please continue[y/n]')*
- reboot: 重启远程主机*reboot()*
- colors: 输入颜色用的
## 常用的命令
- -l: 显示定义好的任务函数名字
- -f: 指定fab入口的文件,默入口文件名为fabile.py
- -g: 指定网关设备,比如堡垒机环境,填写堡垒机ip即可
- -H:指定目标主机, 多台主机用''号分隔
- -P:以异步并行方式运行多个主机任务,默认为串行运行
- -R:指定role角色,以角色名字区分不同业务组设备
- -T: 设定远程主机命令执行超时时间
- -t: 设定设备的连接超时时间
- -w: 当命令执行时效,发出警告,而非默认终止任务
## 常用的装饰器 
- @runs_once: 函数修饰符,标识符的函数只会执行一次,不受多台主机的影响
- @task: 函数修饰符,标识符的函数为fab可调用,非标记的fab不可见,纯属业务逻辑
- @hosts: 定义按照主机来执行*如hosts('user1@hostname','user2@hostname')*
- @roles: 函数修饰符,定义按照角色分组来执行,*如@roles('web','db')*
- @parallel 和@serial: 任务并行或串行执行，如果任务同时被 serial 和 parallel 装饰器装饰，parallel 的优先级更高。
- @with_setting(warn_only=True): warn_only表示是否当在远程机器上执行命令，出现错误时，fabric是否退出。将整个函数封装起来，其效果类似于执行在 settings 上下文管理器中。如果你想要修改函数的设置，但不愿改动其缩进时，它会很有用。
## fabric env 环境变量
- env.host : 主机ip，也可以使用fab选项-H参数来指定
 
- env.roledefs : 角色分组，如：{'web': ['x', 'y'], 'db': ['z']}
 
- env.all_hosts：Default 为 []，由 fab 设置的当前正在执行命令的主机列表。仅用于显示信息。
 
- env.exclude_hosts : Default 为 []，指定一个主机串列表， fab 执行期间会跳过列表中的主机。例：env.exclude_hosts = [ '192.168.1.102' ]
 
- env.port：定义目标主机端口，默认为22。例：env.port = '80'。
 
- env.password : SSH密码，若已经设置好无密码登录，则可以忽略
 
- env.passwords：与 password 功能 一样，区别在于不同主机不同密码的应用场景，需要注意的是，配置passweords时需要配置用户，主机，端口等信息。例：
 
- env.passwords = { 
 'root@192.168.1.104:22':'123',
 'root@192.168.1.86:22':'789',
 'root@222.24.51.147:22':'345643', 
}
 
- env.parallel ：全局并行参数，例 env.parallel = True 。
</code>

## fabric 的简单使用(-)
> Fabric工具提供了一个简单的构建工具fab，其作为Fabric程序的命令行入口，提供了丰富的参数调用。

> fab 默认会读取当前目录叫做fabfile.py名字的文件,这个是默认的, 如果当前目录没有,就去上级目录
也就是父目录中去找这个文件 

~~~python
[root@bdstravel fabric]# cat fabfile.py
#!/usr/bin/env python 

def hello():
        print("hello world")
[root@bdstravel fabric]# fab hello 
hello world

Done.
[root@bdstravel fabric]# 
~~~
> 如果不想以fabfile.py 命名文件的话 ,比如fuck.py 执行的时候如下 

~~~python
[root@bdstravel fabric]# cat fuck.py
#!/usr/bin/env python 
def  fuck():
        print("hello world")
[root@bdstravel fabric]# fab -f fuck.py fuck 
hello world

Done.
~~~
~~~python
@task  #如果定义了task,那么其他的任务也要定以,否则-l的时候任务不显示,提示variable comment 
def mod():                 #本地任务函数
        with lcd('/tmp'):  #‘with’的作用是让后面的表达式语句继承当前状态，即实现‘cd('/tmp') && ls’ 的效果
                run('echo "budongshu" > bds.tmp') 
                run('ls')
[root@bdstravel fabric]# fab -f hostinfo.py mod
[127.0.0.1] Executing task 'mod'
[127.0.0.1] run: echo "budongshu" > bds.tmp
[127.0.0.1] run: ls
[127.0.0.1] out: anaconda-ks.cfg  deploy_git.sh        fabric       install.log.syslog  nohup.out
[127.0.0.1] out: bds.tmp                 dnspod_load_agent.sh  install.log  nginx-1.8.0.tar.gz  shadowsocks-libev
[127.0.0.1] out: 

Done.
~~~


## fabric 的 简单实用(二)
~~~python
[root@bdstravel fabric]# cat hostinfo.py
#!/usr/bin/env python 

from fabric.api import * 
from fabric.colors import * 
import os 

env.user = 'root' 
env.hosts = ['127.0.0.1'] 
env.port = '22'
def hname():
        print('print hostname....') 
        local('hostname')
 
[root@bdstravel fabric]# fab -f hostinfo.py -l
Available commands:

    hname

[root@bdstravel fabric]# fab -f hostinfo.py hname
[127.0.0.1] Executing task 'hname'
print hostname....
[localhost] local: hostname
bdstravel

Done.
 ~~~       
## fabric 简单实用进阶
###不想打印过程
~~~python
[root@bdstravel fabric]# cat hostinfo.py
#!/usr/bin/env python 

from fabric.api import * 
from fabric.colors import * 
import os 

env.user = 'root' 
env.hosts = ['127.0.0.1'] 
env.port = '22'
def hname():
        print(red('print hostname....')) 
        with settings(hide('running'), warn_only=True):
                local('hostname')


[root@bdstravel fabric]# fab -f hostinfo.py hname
[127.0.0.1] Executing task 'hname'
print hostname....        #打印输出的是红色
bdstravel

Done.
~~~
### 对上一步操作做出判断
~~~python
[root@bdstravel fabric]# cat hostinfo.py
#!/usr/bin/env python 
#coding:utf-8
from fabric.api import * 
from fabric.colors import * 
import os 
env.user = 'root' 
env.hosts = ['127.0.0.1'] 
env.port = '22'
def hname():
        print(red('print hostname....')) 
        with settings(hide('running'), warn_only=True):
                res = local('hostname')
                if res.return_code == 0: 
                        print (red('-'*30)) 
                        print (green('打印成功')) 
                        print (red('-'*30))
                else: 
                        print (blue('#'* 30 )) 
                        print (red('打印失败')) 
                        print (blue('#'* 30 ))
~~~
~~~python
[root@bdstravel fabric]# vim hostinfo.py          
[root@bdstravel fabric]# fab -f hostinfo.py  hname
[127.0.0.1] Executing task 'hname'
print hostname....
bdstravel
------------------------------
打印成功
------------------------------

Done
~~~

>   res = local('hostname')  修改为    res = local(red('hostname')) 就是为了让他报出一个错误

~~~python
[root@bdstravel fabric]# fab -f hostinfo.py  hname
[127.0.0.1] Executing task 'hname'
print hostname....
/bin/sh: hostname: command not found

Warning: local() encountered an error (return code 127) while executing 'hostname'

##############################
打印失败
##############################

Done.
~~~ 
## fabric部署lnmp
~~~python
#!/usr/bin/env python 
#coding:utf-8

from fabric.colors import *
from fabric.api import *
"""
自动化部署lnmp环境 
"""
env.roledefs = {
        'web':['128.199.177.154']
        'db':['127.0.0.1']
}
env.passwords = {
        'root@128.199.177.154:22':'centos',
        'root@127.0.0.1:22':'centos'
}
@roles('web')
def webtask():                                  #部署nginx php php-fpm等环境
        print('install nginx php php-fpm...')
        with settings(warn_only=True):
                run('yum install nginx -y')
                run('yum install php-fpm php-mysql php-mbstring php-xml php-mcrypt php-gd -y')
                run('chkconfig --levels 35 php-fpm on')
                run('chkconfig --levels 35 nginx on')
@roles('db')
def dbtask():                                   #部署mysql环境
        print(yellow('install mysql...'))
        with settings(warn_only=True):
                run('yum install mysql-server -y')
                run('chkconfig --levels 35 on')
@roles('web','db')
def default_task():
        print(yellow('install epel ntp ...'))
        with settings(warn_only=True):
                run('wget -P   /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/Centos-6.repo')
                run('yum install ntp')
def deploy():                                     #部署入口
        execute(default_task)
        execute(webtask)
        execute(dbtask)
~~~
