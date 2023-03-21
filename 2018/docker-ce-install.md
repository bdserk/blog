### docker 版本说明
> docker自1.13版本以后发行版本有了很大不同
分为了CE(community edition)社区版和EE(enterprise edition)企业版

- 现在Docker改为基于YY.MM的版本（像Ubuntu） 
- 用户可以选择Stable（发布较慢）或者Edge（发布较快）版本。
- ce由社区维护和提供技术支持，为免费版本
- ee版本为收费版本，由售后团队和技术团队支持技术支持
- 更多的收费标准看docker官网

### docker 安装包说明
- docker.io: is used to be very old version in default ubuntu repo (can skip here)
- docker-engine: is used before release 1.13.x
- docker-ce: since 17.03
### docker-engine 安装
#### yum 安装
```
cat > /etc/yum.repos.d/docker.repo <<EOF

[dockerrepo]

name=Docker Repository

baseurl=https://mirrors.aliyun.com/docker-engine/yum/repo/main/centos/7/

enabled=1

gpgcheck=0

gpgkey=https://yum.dockerproject.org/gpg

EOF
### yum 
yum -y install docker-engine
```
#### 启动管理
```
systemctl start docker
systemctl enable docker
systemctl status docker
```

### docker-ce 安装
#### yum安装 
##### yum 工具
安装所需的软件包 yum-utils提供了yum-config-manager 效用
并device-mapper-persistent-data和lvm2由需要devicemapper存储驱动程序。
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
#####  添加镜像源
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
##### 将软件包添加至本地缓存
```
sudo yum makecache fast
```

##### 安装docker-ce
```
sudo yum install docker-ce
```
##### 启动docker
```
sudo systemctl start docker
```
#### rpm包安装
[rpm下载地址](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)

#### 借助daocloud安装 
[daocloud下载](https://get.daocloud.io/#install-docker)
```
curl -sSL https://get.daocloud.io/docker | sh
```

#### 查看详细信息
docker info
```
[root@c7-vm1_31 ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.11.0-ce
Storage Driver: overlay
 Backing Filesystem: xfs
 Supports d_type: false
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 992280e8e265f491f7a624ab82f3e238be086e49
runc version: 0351df1c5a66838d0c392b4ac4cf9450de844e2d
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-327.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 977.9MiB
Name: c7-vm1_31
ID: BOLL:DHW4:DE7H:Y3TP:WMGG:2X6M:WT3E:RTP5:OWQJ:AXEA:FEF6:RGT5
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false

WARNING: overlay: the backing xfs filesystem is formatted without d_type support, which leads to incorrect behavior.
         Reformat the filesystem with ftype=1 to enable d_type support.
         Running without d_type support will not be supported in future releases.
```
### 配置docker  hub 加速 
> 需要注册daocloud官网
```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://xxxx.m.daocloud.io
```
### docker 拉取镜像
```
docker pull nginx 
```
![image.png](http://upload-images.jianshu.io/upload_images/1542757-4b22f6aff03063c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)
```
docker run hello-world
``` 
![image.png](http://upload-images.jianshu.io/upload_images/1542757-30654bcdbcc30415.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)




