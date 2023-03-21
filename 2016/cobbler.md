### 自动化工具分为三大类
* 预备类（Os Provisioning）
  +  PXE  
  +  cobbler
* 配置管理类（Os config & Devops ）
 + cfengine 
 + chef
 + puppet
 + saltstack
 + func
 + fabric
 + ansible 
* 监控类（Mointor）
  + Cacti
  + Nagios Core
  + Zabbix
  + Zenoss core

## 网站灰度发布（依赖于前端的lb实现）
* 关闭Directory上一批服务器
* 关闭这些服务器要更新的应用
* 更新webapp代码至目标主机
* 启动目标应用 ，Dirtectory启动这批服务器

## 发布大致流程
* 代码控制（csv，svn，git）检出要发布的代码，发布至预发布服务器上
* 预发布服务器： 预发布服务器跟线上服务器环境一致，但不加入线上机器中，实施详细的测试
* 自动化测试，使用webapp自动化测试工具（如thoughworks开发的selenium）可以进行完整的代码，浏览器兼容性的测试 
* 自动化灰度发布，线上批量分批次更新代码

## 网站运行监控
* 监控数据采集，用户行为日志，服务器性能监控，运行数据报告
* 监控管理 异常报警，失败转移，自动优雅降级

## 前言
   运维自动化在生产环境中占据着举足轻重的地位，尤其是面对几百台，几千台甚至几万台的服务器时，仅仅是安装操作系统，如果不通过自动化来完成，根本是不可想象的。记得前面我们探究了基于PXE实现系统全自动安装，但PXE同时只能提供单一操作系统的批量部署，面对生产环境中不同服务器的需求，该如何实现批量部署多版本的操作系统呢？Cobbler便可以的满足这一实际需求，本文带来的是基于Cobbler实现多版本操作系统批量部署。
## cobbler 简介
Cobbler是一款自动化操作系统部署的实现工具，由Python语言开发，是对PXE的二次封装。融合多种特性，提供了CLI和Web的管理形式。同时，Cobbler也提供了API接口，方便二次开发使用。它不仅可以安装物理机，同时也支持kvm、xen虚拟化、Guest OS的安装。另外，它还能结合Puppet等集中化管理软件，实现自动化管理。

## 组件
Cobbler的各主要组件间关系如图所示
![image](http://upload-images.jianshu.io/upload_images/1542757-485a1a4a6e3620c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## cobbler 服务集成
* pxe 服务
* DHCP
* Rsync
* Http
* DNS
* Kickstart
* IPMI  电源管理 

## cobbler 设计方式
* 发行版（distro） ：表示一个操作系统，它承载了内核和initrd的信息，以及内核等其他数据
* 存储库 （repository）：保存了一个yum或者rsync存储库的镜像信息
* 配置文件（profile）：包含了一个发行版（distro），一个kickstart文件以及可能的存储库（repository），还包含了更多的内核参数等其他数据
* 系统（system）：表示要配给的机器，它包含了一个配置文件或一个镜像，还包含了ip和mac地址，电源管理（地址,凭据,类型）以及更为专业的数据信息
* 镜像（image）：可替换一个包含不属于此类别的文件的发行版对象（eg: 无法作为内核和initrd的对象）
*以上各个组件中， 发行版，存储库， 配置文件为必须配置项
只有在虚拟环境中，必须要用cobbler来引导虚拟机启动时候，才会用到系统组件
但事实上，在生产环境中需要大量的虚拟机实例的话，通常利用openstack等来实现虚拟机节点*

## cobbler 运行流程
- dhcp 
- client: 从dhcp中获取地址，访问next_server的ip地址
- next_server : 获取启动内核，initrd等文件
- tftp: pxe引导文件，启动cobbler选择界面
- kickstart: 确定加载项，根据nfs，http，tfp等共享获取资源

## cobbler units
- cobbler
- cobbler-web

## 配置cobbler 步骤
1 安装cobbler，依据cobbler check检查结果，对setting主配置文件，进行相关的修正配置
2 启动相关的http，cobbler服务，使用cobbler sync同步设置
3 配置cobbler 所依赖的包
         * dhcp  
         * dns
         * rsync
         * tftp
4 配置cobbler组件
**针对步骤3 ，需要： 
  1 选定要使用的程序，选其一管理即可
  2 确定是独立管理这些服务，还有由cobbler代为管理
**
*注意事项：
   cobbler本身是不提供对应的服务程序的，因此还是需要安装对应的程序服务的rpm包，并保证其开启动的状态，由cobbler管理这些服务 *

## 安装cobbler 
###安装epel源
```python
[root@kvm ~]# yum install -y wget 
[root@kvm ~]# wget -O /etc/yum.repos.d/epel.repo   http://mirrors.aliyun.com/repo/epel-6.repo
```
## 设置ip转发
```python
[root@kvm ~]# echo 1 > /proc/sys/net/ipv4/ip_forward 
[root@kvm ~]# sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/' /etc/sysctl.conf
[root@kvm ~]# sysctl -p
```
### 安装cobbler
```python
[root@kvm ~]# yum -y install cobbler dhcp httpd xinetd tftp-server syslinux pykickstart xinetd rsync cobbler-web
```
### cobbler 各种配置目录说明
---
**配置文件目录 /etc/cobbler**
- /etc/cobbler/settings : cobbler 主配置文件
- /etc/cobbler/iso/: iso模板配置文件
- /etc/cobbler/pxe: pxe模板文件
- /etc/cobbler/power: 电源配置文件
- /etc/cobbler/user.conf: web服务授权配置文件
- /etc/cobbler/users.digest: web访问的用户名密码配置文件
- /etc/cobbler/dhcp.template : dhcp服务器的的配置末班
- /etc/cobbler/dnsmasq.template : dns服务器的配置模板
- /etc/cobbler/tftpd.template : tftp服务的配置模板
- /etc/cobbler/modules.conf : 模块的配置文件

**数据目录** 
- /var/lib/cobbler/config/: 用于存放distros，system，profiles 等信息配置文件
- /var/lib/cobbler/triggers/: 用于存放用户定义的cobbler命令
- /var/lib/cobbler/kickstart/: 默认存放kickstart文件
- /var/lib/cobbler/loaders/: 存放各种引导程序

**镜像目录** 
- /var/www/cobbler/ks_mirror/: 导入的发行版系统的所有数据
- /var/www/cobbler/images/ : 导入发行版的kernel和initrd镜像用于远程网络启动
- /var/www/cobbler/repo_mirror/: yum 仓库存储目录

**日志目录**
- /var/log/cobbler/installing: 客户端安装日志
- /var/log/cobbler/cobbler.log : cobbler日志 

### cobbler commands
* import
* sync
* reposync
* build iso  （使用发行版，配置文件，制作系统镜像）
* command line search
* replication
* valication kickstart

###动态更新配置

对于Cobbler2.4来说，有一个重要的功能，就是让你可以不需要手工去编辑setting配置文件，直接使用命令去修改，默认这个功能是不启用，你需要启用。
```python
[root@kvm cobbler]# cp settings  settings.bak 
[root@kvm cobbler]# sed -i 's/^[[:space:]]\+/ /' /etc/cobbler/settings 
[root@kvm cobbler]# sed -i 's/allow_dynamic_settings: 0/allow_dynamic_settings: 1/g' /etc/cobbler/settings
[root@kvm cobbler]# /etc/init.d/cobblerd restart 
Stopping cobbler daemon:                                   [  OK  ]
Starting cobbler daemon:                                   [  OK  ]
```
*建议采用修改/etc/cobbler/settings配置文件的方式修改相关配置选项*
### 检查需要安装的配置
```python
[root@note1 ~]# cobbler check
The following are potential configuration items that you may want to fix:


1 : The 'server' field in /etc/cobbler/settings must be set to something other than localhost, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it.

2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than 127.0.0.1, and should match the IP of the boot server on the PXE network.

3 : change 'disable' to 'no' in /etc/xinetd.d/tftp

4 : some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.

5 : change 'disable' to 'no' in /etc/xinetd.d/rsync

6 : since iptables may be running, ensure 69, 80/443, and 25151 are unblocked

7 : reposync is not installed, need for cobbler reposync, install/upgrade yum-utils?

8 : debmirror package is not installed, it will be required to manage debian deployments and repositories

9 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one

10 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them
```
*我这里人品不好，出现了十个，真够多的*

**解决1,2问题**
```python
1: [root@kvm cobbler]# cobbler setting edit --name=server --value=10.10.10.10
2: [root@kvm cobbler]# cobbler setting edit --name=next_server --value=10.10.10.10
```
**解决3,5问题**
```python
[root@note1 cobbler]# chkconfig tftp on 
[root@note1 cobbler]# chkconfig rsync on
[root@note1 cobbler]# /etc/init.d/xinetd restart
```
**解决4问题**
>下载启动菜单

```python
[root@note1 cobbler]# cobbler get-loaders
```
**解决6问题**
配置防火墙，我的防火墙是开启的，配置完策略后，但还是报，但是后期不耽误正常配置，不知道什么原因
```python
iptables -A INPUT -m state --state NEW  -m tcp -p tcp -m multiport --dports 80,443,88,25151 -j ACCEPT
iptables -A INPUT -m state --state NEW  -m udp -p udp -m multiport --dports 53,67,68,25252 -j ACCEPT
iptables -A INPUT -m state --state NEW  -m udp -p udp --dport 69 -j ACCEPT
```
**7问题**
配置管理repo仓库的，可以忽略
**8问题**
跟debian系统有关，如果有需要装一下即可，否则可以忽略
```python
yum -y install debmirror
```
**解决9问题**
```python
[root@note1 loaders]# openssl passwd -1 -salt `openssl rand -hex 4` "budongshu"
$1$557d907c$AmKQun9Jxitt1D6aQ8DUC.
[root@kvm cobbler]# cobbler setting  edit --name=default_password_crypted --value="$1$557d907c$AmKQun9Jxitt1D6aQ8DUC."   #如果命令不生效，去手动配置文件修改
```
**解决10问题**
```python
yum install -y cman  　　　#安装电源管理工具
```
**防止误重装系统，选项pxe_just_one** 
```python
[root@note1 cobbler]# cobbler setting edit --name=pxe_just_once --value=1
```
### 配置rsync，tftp 服务 由cobbler管理
* 默认情况下，cobbler安装完后，会自己去管理tftp服务器，因manage_tftp
和managed_tftpd 的值默认为1 
* 配置tftp rsync 服务，保证服务已经安装，并且设置为开机自动启动
* 需要保证xinetd服务为开机自动启动状态，因rsync，tftp 服务由xinetd服务统一管理

```python
前面执行chkconfig rysnc on 和chkconfig tftp on,diable 应该就是yes了，配置文件/etc/xinetd.d/rsync
[root@note1 cobbler]# chkconfig  --level 35 xinetd on
```

### 配置dhcp服务由cobbler来管理（这里使用cobbler管理dhcp器，也可以使用dnsmasq来管理）
> 配置dhcpd配置文件

```python
[root@note1 cobbler]# vim /etc/cobbler/dhcp.template
#其他暂时不需要动，只修改下面的几个内容
subnet 10.10.10.0 netmask 255.255.255.0 {
     option routers             10.10.10.10;
     option domain-name-servers      114.114.114.114 ;
     option subnet-mask         255.255.255.0;
     range dynamic-bootp        10.10.10.20 10.10.10.50;
     default-lease-time         21600;
     max-lease-time             43200;
     next-server                $next_server;
```
>此时的dhcpd的配置就被cobbler覆盖 ，由cobbler来管理配置文件，截取的一部分，后面还有内容

```python
[root@note1 cobbler]# vim  /etc/dhcp/dhcpd.conf 
# ******************************************************************
# Cobbler managed dhcpd.conf file
# generated from cobbler dhcp.conf template (Sun Jan 31 15:47:49 2016)
# Do NOT make changes to /etc/dhcpd.conf. Instead, make your changes
# in /etc/cobbler/dhcp.template, as /etc/dhcpd.conf will be
# overwritten.
# *****************************************************************
ddns-update-style interim;
allow booting
allow bootp
ignore client-updates;
set vendorclass = option vendor-class-identifier
option pxe-system-type code 93 = unsigned integer 16;
subnet 10.10.10.0 netmask 255.255.255.0 {
--------
```
## 同步cobbler
```python
[root@kvm cobbler]# service cobblerd restart 
Stopping cobbler daemon:                                   [  OK  ]
Starting cobbler daemon:                                   [  OK  ]
[root@kvm cobbler]# cobbler sync
task started: 2015-11-06_094656_sync
task started (id=Sync, time=Fri Nov  6 09:46:56 2015)
running pre-sync triggers
cleaning trees
removing: /var/lib/tftpboot/grub/images
copying bootloaders
trying hardlink /var/lib/cobbler/loaders/pxelinux.0 -> /var/lib/tftpboot/pxelinux.0
trying hardlink /var/lib/cobbler/loaders/menu.c32 -> /var/lib/tftpboot/menu.c32
trying hardlink /var/lib/cobbler/loaders/yaboot -> /var/lib/tftpboot/yaboot
trying hardlink /usr/share/syslinux/memdisk -> /var/lib/tftpboot/memdisk
trying hardlink /var/lib/cobbler/loaders/grub-x86_64.efi -> /var/lib/tftpboot/grub/grub-x86_64.efi
trying hardlink /var/lib/cobbler/loaders/grub-x86.efi -> /var/lib/tftpboot/grub/grub-x86.efi
copying distros to tftpboot
copying images
generating PXE configuration files
generating PXE menu structure
rendering TFTPD files
generating /etc/xinetd.d/tftp
cleaning link caches
running post-sync triggers
running python triggers from /var/lib/cobbler/triggers/sync/post/*
running python trigger cobbler.modules.sync_post_restart_services
running shell triggers from /var/lib/cobbler/triggers/sync/post/*
running python triggers from /var/lib/cobbler/triggers/change/*
running python trigger cobbler.modules.scm_track
running shell triggers from /var/lib/cobbler/triggers/change/*
*** TASK COMPLETE ***
```
>重启

```python
[root@kvm cobbler]# service cobblerd restart 
Stopping cobbler daemon:                                       [  OK  ]
Starting cobbler daemon:                                         [  OK  ]
```
>再次检查

```python
[root@kvm cobbler]# cobbler check     #除了上边刚才必须修改的，以下的错误，不耽误正常配置
```
### 启动相关服务
```python
[root@kvm ~]# chkconfig httpd on
[root@kvm ~]# chkconfig cobblerd on 
[root@kvm Data]# chkconfig tftp on 
[root@kvm Data]# chkconfig rsync on    
[root@kvm Data]# chkconfig xinetd on  
[root@kvm Data]# service xinetd start
[root@kvm Data]# service cobblerd start 
[root@kvm Data]# service httpd start 
```
### 编写启动脚本
```python
cat >>/etc/init.d/cobbler<<EOF
#!/bin/bash
# chkconfig: 345 80 90
# description:cobbler
case \$1 in
start)
/etc/init.d/httpd start
/etc/init.d/xinetd start
/etc/init.d/dhcpd start
/etc/init.d/cobblerd start  ;;
stop)
/etc/init.d/httpd stop
/etc/init.d/xinetd stop
/etc/init.d/dhcpd stop
/etc/init.d/cobblerd stop    ;;
restart)
/etc/init.d/httpd restart
/etc/init.d/xinetd restart
/etc/init.d/dhcpd restart
/etc/init.d/cobblerd restart   ;;
status)
/etc/init.d/httpd status
/etc/init.d/xinetd status
/etc/init.d/dhcpd status
/etc/init.d/cobblerd status   ;;
sync)
cobbler sync   ;;
*)
echo "Input error,please in put 'start|stop|restart|status|sync'!"
exit 2  ;;
esac
EOF
# chmod +x /etc/init.d/cobbler
# chkconfig cobbler on
```
### 配置命令
```python
[root@linux-node1 ~]# cobbler
usage
=====
cobbler <distro|profile|system|repo|image|mgmtclass|package|file> ... 
 [add|edit|copy|getks*|list|remove|rename|report] [options|--help]
cobbler <aclsetup|buildiso|import|list|replicate|report|reposync|sync|validateks|version|signature|get-loaders|hardlink> [options|--help]
[root@linux-node1 ~]# cobbler import --help # 导入镜像
Usage: cobbler [options]
Options:
 -h, --help show this help message and exit
 --arch=ARCH OS architecture being imported
 --breed=BREED the breed being imported
 --os-version=OS_VERSION
 the version being imported
 --path=PATH local path or rsync location
 --name=NAME name, ex 'RHEL-5'
 --available-as=AVAILABLE_AS
 tree is here, don't mirror
 --kickstart=KICKSTART_FILE
 assign this kickstart file
 --rsync-flags=RSYNC_FLAGS
 pass additional flags to rsync
cobbler check 核对当前设置是否有问题
cobbler list 列出所有的cobbler元素
cobbler report 列出元素的详细信息
cobbler sync 同步配置到数据目录,更改配置最好都要执行下
cobbler reposync 同步yum仓库
cobbler distro 查看导入的发行版系统信息
cobbler system 查看添加的系统信息
cobbler profile 查看配置信息
```
### 导入系统到cobbler
> centos6.5

```python
[root@kvm cobbler]# mount /dev/cdrom  /mnt  
[root@note1 cobbler]# cobbler import --path=/mnt/ --name=Centos-6.5-x86_64 --arch=x86_64
[root@note1 cobbler]# cobbler distro report --name=Centos-6.5-x86_64 
Name                           : Centos-6.5-x86_64
Architecture                   : x86_64
TFTP Boot Files                : {}
Breed                          : redhat
Comment                        : 
Fetchable Files                : {}
Initrd                         : /var/www/cobbler/ks_mirror/Centos-6.5-x86_64/images/pxeboot/initrd.img
Kernel                         : /var/www/cobbler/ks_mirror/Centos-6.5-x86_64/images/pxeboot/vmlinuz
Kernel Options                 : {}
Kernel Options (Post Install)  : {}
Kickstart Metadata             : {'tree': 'http://@@http_server@@/cblr/links/Centos-6.5-x86_64'}
Management Classes             : []
OS Version                     : rhel6
Owners                         : ['admin']
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Template Files                 : {}
```
>centos7

```python
[root@note1 ~]# umount /mnt
[root@note1 cobbler]# cobbler import --path=/mnt/ --name=Centos-7-x86_64 --arch=x86_64
[root@note1 kickstarts]# cobbler distro report --name=Centos-7-x86_64
Name : Centos-7-x86_64
Architecture : x86_64
TFTP Boot Files : {}
Breed : redhat
Comment : 
Fetchable Files : {}
Initrd : /var/www/cobbler/ks_mirror/Centos-7-x86_64/images/pxeboot/initrd.img
Kernel : /var/www/cobbler/ks_mirror/Centos-7-x86_64/images/pxeboot/vmlinuz
Kernel Options : {}
Kernel Options (Post Install) : {}
Kickstart Metadata : {'tree': 'http://@@http_server@@/cblr/links/Centos-7-x86_64'}
Management Classes : []
OS Version : rhel7
Owners : ['admin']
Red Hat Management Key : <<inherit>>
Red Hat Management Server : <<inherit>>
Template Files : {}
```
> 查看

```python
[root@note1 cobbler]# cobbler distro list
   Centos-6.5-x86_64
   Centos-7-x86_64
```
### 修改默认ks文件
> 配置centos6.5

```python
# kickstart template for Fedora 8 and later.
# (includes %end blocks)
# do not use with earlier distros

#platform=x86, AMD64, or Intel EM64T
# Install OS instead of upgrade
install
# Use network installation
url --url=$tree
# Use text mode install
text
# System language
lang en_US
# System keyboard
keyboard us
# Clear the Master Boot Record
zerombr
# System bootloader configuration
bootloader --location=mbr
# System timezone
timezone  America/Shanghai
# System authorization information
auth  --useshadow  --enablemd5
#Root password
rootpw --iscrypted $default_password_crypted
# Network information
$SNIPPET('network_config')
# Partition clearing information
clearpart --all --initlabel
# Allow anaconda to partition the system as needed
part /boot --fstype ext4 --size 200 
part swap --fstype swap --size 2000  
part /  --fstype ext4 --size 20000 
part /data --fstype ext4 --size 1 --grow 
# Firewall configuration
firewall --enabled
# Run the Setup Agent on first boot
firstboot --disable
# SELinux configuration
selinux --disabled
# If any cobbler repo definitions were referenced in the kickstart profile, include them here.
$yum_repo_stanza
# Reboot after installation
reboot

# Do not configure the X Window System
skipx

%pre
$SNIPPET('log_ks_pre')
$SNIPPET('kickstart_start')
$SNIPPET('pre_install_network_config')
# Enable installation monitoring
$SNIPPET('pre_anamon')
%end

%packages
$SNIPPET('func_install_if_enabled')
lrzsz 
tree
wget
curl
openssh 
openssl 

%end

%post --nochroot
$SNIPPET('log_ks_post_nochroot')
%end

%post
$SNIPPET('log_ks_post')
# Start yum configuration
$yum_config_stanza 

# End yum configuration
$SNIPPET('post_install_kernel_options')
$SNIPPET('post_install_network_config')
$SNIPPET('func_register_if_enabled')
$SNIPPET('cobbler_register')
# Enable post-install boot notification
$SNIPPET('post_anamon')
# Start final steps
$SNIPPET('kickstart_done')
# End final steps

mkdir /root/backup 
sed -i "s/#UseDNS yes/UseDNS no/"  /etc/ssh/sshd_config
sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
yum -y install git lrzsz

%end
```
**到此centos6.5系统就可以装机了**

> 配置centos7

```python
[root@note1 kickstarts]# cat CentOS-7-x86_64.cfg 
#obbler for Kickstart Configurator for CentOS 7.1 by yao zhang
install
url --url=$tree  
text
lang en_US.UTF-8
keyboard us
zerombr
bootloader --location=mbr 
# Network information
$SNIPPET('network_config')
timezone --utc Asia/Shanghai
authconfig --enableshadow --passalgo=sha512
rootpw  --iscrypted $default_password_crypted
clearpart --all --initlabel
part /boot --fstype xfs --size 500  
part swap --size 2000
part / --fstype xfs --size 20000 
part /data --fstype xfs --size 30000 
firstboot --disable
selinux --disabled
firewall --disabled
logging --level=info
reboot
%pre
$SNIPPET('log_ks_pre')
$SNIPPET('kickstart_start')
$SNIPPET('pre_install_network_config')
# Enable installation monitoring
$SNIPPET('pre_anamon')
%end
%packages
@base
@compat-libraries
@debugging
@development
tree
nmap
sysstat
lrzsz
dos2unix
telnet
iptraf
ncurses-devel
openssl-devel
zlib-devel
OpenIPMI-tools
screen
%end
%post
systemctl disable postfix.service
%end
```
### 修改centos7 网卡label
```python
# 修改安装系统的内核参数，在CentOS7系统有一个地方变了，就是网卡名变成eno16777736这种形式，但是为了运维标准化，
# 我们需要将它变成我们常用的eth0，因此使用下面的参数。但要注意是CentOS7才需要下面的步骤，CentOS6不需要。
[root@note1 kickstarts]# cobbler profile edit --name=CentOS-7.1-x86_64 --kopts='net.ifnames=0 biosdevname=0'
[root@note1 kickstarts]# cobbler profile report --name=CentOS-7-x86_64 
Name                           : CentOS-7-x86_64
TFTP Boot Files                : {}
Comment                        : 
DHCP Tag                       : default
Distribution                   : Centos-7-x86_64
Enable gPXE?                   : 0
Enable PXE Menu?               : 1
Fetchable Files                : {}
Kernel Options                 : {'biosdevname': '0', 'net.ifnames': '0'}
Kernel Options (Post Install)  : {}
Kickstart                      : /var/lib/cobbler/kickstarts/CentOS-7-x86_64.cfg
Kickstart Metadata             : {}
Management Classes             : []
Management Parameters          : <<inherit>>
Name Servers                   : []
Name Servers Search Path       : []
Owners                         : ['admin']
Parent Profile                 : 
Internal proxy                 : 
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Repos                          : []
Server Override                : <<inherit>>
Template Files                 : {}
Virt Auto Boot                 : 1
Virt Bridge                    : xenbr0
Virt CPUs                      : 1
Virt Disk Driver Type          : raw
Virt File Size(GB)             : 5
Virt Path                      : 
Virt RAM (MB)                  : 512
Virt Type                      : kvm
```
> 查看

```python
[root@note1 kickstarts]# cobbler profile report  Centos-7-x86_64
[root@note1 kickstarts]# cobbler profile report Centos-6.5-x86_64
[root@note1 kickstarts]# cobbler list
distros:
   Centos-6.5-x86_64
   Centos-7-x86_64
profiles:
   Centos-6.5-x86_64
   Centos-7-x86_64
systems:
   budongshu
repos:
images:
mgmtclasses:
packages:
files:
```
> 同步

```python
 [root@note1 kickstarts]# cobbler sync
```
### 配置repo仓库
> 配置本地yum仓库  (选配,可以不配置)

```python
[root@localhost ~]# mkdir /tmp/rpms
[root@localhost ~]# createrepo /tmp/rpms  #放入rpm包,执行此步骤
[root@localhost ~]# cobbler repo add --mirror=/tmp/rpms --name=local
[root@localhost ~]# cobbler reposync
```

> 配置本地epel仓库(选配,可以不配置)

```python
[root@localhost ~]# cobbler repo add --mirror=http://mirrors.aliyun.com/epel/6/x86_64/ --name=epel
[root@localhost ~]# cobbler reposync --tries=3 --no-fail  #同步epel仓库到
本地,需要较长时间
```
> 查看已添加的repo(选配,可以不配置)

```python
[root@localhost ~]# cobbler repo list
   epel
   local
```
>添加repo到profile(选配,可以不配置)

```python
[root@localhost ~]# cobbler profile edit --name=Centos-6.5-x86_64 --repos="epel local"
[root@localhost ~]# cobbler sync
```
### 绑定mac地址 ,实现开机自动选择

![image](http://upload-images.jianshu.io/upload_images/1542757-a966834307a484b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>配置绑定mac地址和IP地址 ,开机自动选择

```python
[root@note1 kickstarts]# cobbler system add --name=budongshu --mac=00:0C:29:48:1D:75 --profile=Centos-7-x86_64  \
--ip-address=10.10.10.23 --subnet=255.255.255.0 --gateway=10.10.10.10 --interface=eth0 \
--static=1 --hostname=budongshu --name-servers="114.114.114.114 8.8.8.8"

 [root@note1 kickstarts]# cobbler sync
```
**到此cento7系统的也可以装机了，并且绑定了mac地址，固定了ip地址**

---
### web界面配置
cobbler-web支持多种认证方式，如authn_configfil、authn_ldap或authn_pam等，下面我们基于authn_pam做认证
>修改认证方式

```python
[root@note1 web]# vim /etc/cobbler/modules.conf 
[authentication]
module = authn_pam
```

>添加系统用户

```python
[root@note1 web]# useradd cobbler 
[root@note1 web]# echo "cobbler" | passwd --stdin cobbler
```
>添加用户到管理组

```python
[root@note1 web]# vim /etc/cobbler/modules.conf
[admins]
admin = "cobbler"
```
>重启服务

```python
[root@note1 web]# service cobblerd restart 
Stopping cobbler daemon:                                   [  OK  ]
Starting cobbler daemon:                                   [  OK  ]
[root@note1 web]# service httpd restart
Stopping httpd:                                             [  OK  ]
Starting httpd:                                             [  OK  ]
```

![2016-02-06_013919.png](http://upload-images.jianshu.io/upload_images/1542757-3484057f801cfb15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2016-02-06_014051.png](http://upload-images.jianshu.io/upload_images/1542757-cf38839c095ea589.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



