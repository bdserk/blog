
## 环境
关闭防火墙，selinux,基于主机名访问（dns或者dns）,同步时间（时间不一致，集群运行异常，如不能启动）,关闭swap分区
```
#hosts
10.1.101.202  inspur2.ops.bds.bj1 inspur2
10.1.101.203  inspur2.ops.bds.bj1 inspur3
```
### 系统初始化
```
# 关掉 selinux
setenforce  0 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux
# 关掉防火墙 
systemctl stop firewalld
systemctl disable firewalld
#关闭swap
swapoff -a
#开启转发的参数，根据实际报错情况开启，一般有如下三项
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
#加载内核配置
sysctl --system   /etc/sysctl.d/k8s.conf
# 加载ipvs相关内核模块
# 如果重新开机，需要重新加载
   modprobe ip_vs
   modprobe ip_vs_rr
   modprobe ip_vs_wrr
   modprobe ip_vs_sh
   modprobe nf_conntrack_ipv4
   lsmod | grep ip_vs
```

## master 配置
### docker镜像

```
cd /etc/yum.repos.d/
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
### 配置docker镜像加速
```
mkdir -p /etc/docker
vim /etc/docker/daemon.json
{
"registry-mirrors": ["http://08b61f14.m.daocloud.io"],
 "exec-opts": ["native.cgroupdriver=systemd"]
}
```
阿里云加速
[https://cr.console.aliyun.com/undefined/instances/mirrors](https://cr.console.aliyun.com/undefined/instances/mirrors)
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://27j02xby.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 配置拉取gcr.io镜像
1 通过配置代理来下载k8s镜像
```
vim /usr/lib/systemd/system/docker.service
Environment="HTTPS_PROXY=http://www.ik8s.io:10080"
Environment="HTTP_PROXY=http://www.ik8s.io:10080"
Environment="NO_PROXY=127.0.0.0/8,172.20.0.0/16"
#保存退出后，执行
systemctl  daemon-reload
systemctl  enable docker
systemctl  start  docker 
```
2 技术大牛github拉取镜像

[github地址](https://github.com/anjia0532/gcr.io_mirror)
```
vim pullimages.sh
#!/bin/bash
images=(kube-proxy-amd64:v1.11.2 kube-scheduler-amd64:v1.11.2 kube-controller-manager-amd64:v1.11.2 kube-apiserver-amd64:v1.11.2
etcd-amd64:3.2.18 coredns:1.1.3 pause:3.1 )
for imageName in ${images[@]} ; do
docker pull anjia0532/google-containers.$imageName
docker tag anjia0532/google-containers.$imageName k8s.gcr.io/$imageName
docker rmi anjia0532/google-containers.$imageName
done 
```


### k8s repo镜像
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
### install kubeadm
```
yum -y install kubeadm kubelet kubectl
```
### kubeadm config 
```
vim  /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"

systemctl enable kubelet.service
systemctl start kubelet
```

### kubelet  error log

`注意，此时启动会出现报错，查看/var/log/messages的日志`
```
tail -f /var/log/messages

如果出现如下的报错

failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/kubelet/config.yaml: no such file or directory

忽略以上的报错，设置为开机自启动即可，因为此时的配置还没初始化完成，所以此时不能启动kubelet,等后续kubeadm启动成功后再查看
```
### swap 分区注意

`注意，需要关闭swap分区，或者在如下的配置文件里修改，表示添加而且的`

```
启动选项
vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```
`建议执行 swapoff -a 关闭swap分区，不用配置上述选项`

### k8s 初始化
master节点上用kubeadm命令来初始化集群 这个时候会拉取gcr.io镜像时间会比较长
```
kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```
因为我们这里选择flannel作为Pod网络插件，所以需要制定--pod-netwrok-cidr=10.244.0.0/16 

初始化过程会生成相关的证书，kubeconfig文件，bootstraptoken等等
后面是使用kubeadm join往集群中添加节点时用到的命令，这个命令后面的
--token和--discovery-token-ca-cert-hash一定要保留备份好，后面添加节点的时候会使用到，如果丢失也可以通过命里找回，但是比较麻烦。

### 配置使用kubectl访问集群

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 查看master k8s状态
```
#查看组件的健康状态
[root@inspur2 ~]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
#查看集群中node
[root@inspur2 ~]# kubectl get nodes
NAME                    STATUS    ROLES     AGE       VERSION
inspur2.ops.haodf.bj1   Ready     master    16h       v1.11.3
#查看nodespace名称空间
[root@inspur2 ~]# kubectl get ns
NAME          STATUS    AGE
default       Active    16h
kube-public   Active    16h
kube-system   Active    16h
```
### install Pod network
##### master节点安装下载
```

[root@inspur2 ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
[root@inspur2 ~]# kubectl apply -f  kube-flannel.yml
clusterrole.rbac.authorization.k8s.io "flannel" created
clusterrolebinding.rbac.authorization.k8s.io "flannel" created
serviceaccount "flannel" created
configmap "kube-flannel-cfg" created
daemonset.extensions "kube-flannel-ds" created
```
##### 多网卡配置
如果你的节点有多个网卡的话,需要在kube-flannel.yml 中使用 --iface 参数指定集群主机内网网卡的名称，否则可能会出现dns无法解析。 flanneld 启动参数加上 --iface=
```
args:
- --ip-masq
- --kube-subnet-mgr
- --iface=eth0
```
### 查看gcr.io镜像和fannel镜像
```
[root@inspur2 ~]# docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy-amd64                v1.11.1             d5c25579d0ff        8 weeks ago         97.8MB
k8s.gcr.io/kube-scheduler-amd64            v1.11.1             272b3a60cd68        8 weeks ago         56.8MB
k8s.gcr.io/kube-controller-manager-amd64   v1.11.1             52096ee87d0e        8 weeks ago         155MB
k8s.gcr.io/kube-apiserver-amd64            v1.11.1             816332bd9d11        8 weeks ago         187MB
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        3 months ago        45.6MB
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        5 months ago        219MB
quay.io/coreos/flannel                     v0.10.0-amd64       f0fad859c909        7 months ago        44.6MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        8 months ago        742kB

[root@inspur2 ~]# kubectl get pods -n kube-system
NAME                                            READY     STATUS    RESTARTS   AGE
coredns-78fcdf6894-m9kwr                        1/1       Running   1          16h
coredns-78fcdf6894-tgjn6                        1/1       Running   1          16h
etcd-inspur2.ops.haodf.bj1                      1/1       Running   1          16h
kube-apiserver-inspur2.ops.haodf.bj1            1/1       Running   1          16h
kube-controller-manager-inspur2.ops.haodf.bj1   1/1       Running   1          16h
kube-flannel-ds-amd64-d8vxk                     1/1       Running   5          16h
kube-proxy-24gvj                                1/1       Running   1          16h
kube-scheduler-inspur2.ops.haodf.bj1            1/1       Running   1          16h

```
### 如果集群遇到问题，使用下面命令来重置

```
kubeadm reset
ifconfig cni0 down && ip link delete cni0
ifconfig flannel.1 down && ip link delete flannel.1
rm -rf /var/lib/cni/
```
## node节点操作

### yum仓库准备好后，在以下的两个节点上执行安装如下包，
```
yum -y install kubeadm kubelet kubectl docker-ce
```
### 系统初始化
```
# 关掉 selinux
setenforce  0 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux
# 关掉防火墙 
systemctl stop firewalld
systemctl disable firewalld
#关闭swap
swapoff -a
#开启转发的参数，根据实际报错情况开启，一般有如下三项
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
#加载内核配置
sysctl --system   /etc/sysctl.d/k8s.conf
```
### start docker

```
# 设置开启自启动
systemctl enable docker.service
systemctl enable kubelet.service
# 
systemctl start docker
```
### 把配置文件从master拷贝到node节点 保持配置一致

```
scp /usr/lib/systemd/system/docker.service 10.1.101.203:/usr/lib/systemd/system/docker.service
scp /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 10.1.101.203:/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
scp /etc/sysconfig/kubelet 10.1.101.203:/etc/sysconfig/
```

### node节点加入master
```
kubeadm join 10.1.101.202:6443 --token gt8abw.jaaivoggg2imqnld --discovery-token-ca-cert-hash sha256:2220de69ce7084d672dd1f282246a76c4a5bb04661b2ada23a3014e203eb14b3
```
##### node执行上面命令成功后，提示信息如下
```
......
This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```
*看到上面信息代表发布成功*
##### node 节点容器信息
```
[root@inspur3 ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy-amd64   v1.11.1             d5c25579d0ff        8 weeks ago         97.8MB
quay.io/coreos/flannel        v0.10.0-amd64       f0fad859c909        7 months ago        44.6MB
k8s.gcr.io/pause              3.1                 da86e6ba6ca1        8 months ago        742kB
```
### 验证集群
```
[root@inspur2 ~]# kubectl get pod -n kube-system -o wide
NAME                                            READY     STATUS    RESTARTS   AGE       IP             NODE
coredns-78fcdf6894-m9kwr                        1/1       Running   1          16h       10.244.0.5     inspur2.ops.haodf.bj1
coredns-78fcdf6894-tgjn6                        1/1       Running   1          16h       10.244.0.4     inspur2.ops.haodf.bj1
etcd-inspur2.ops.haodf.bj1                      1/1       Running   1          16h       10.1.101.202   inspur2.ops.haodf.bj1
kube-apiserver-inspur2.ops.haodf.bj1            1/1       Running   1          16h       10.1.101.202   inspur2.ops.haodf.bj1
kube-controller-manager-inspur2.ops.haodf.bj1   1/1       Running   1          16h       10.1.101.202   inspur2.ops.haodf.bj1
kube-flannel-ds-amd64-d8vxk                     1/1       Running   7          16h       10.1.101.202   inspur2.ops.haodf.bj1
kube-flannel-ds-amd64-lh2jf                     1/1       Running   0          5m        10.1.101.203   inspur3.ops.haodf.bj1
kube-proxy-24gvj                                1/1       Running   1          16h       10.1.101.202   inspur2.ops.haodf.bj1
kube-proxy-n29ch                                1/1       Running   0          5m        10.1.101.203   inspur3.ops.haodf.bj1
kube-scheduler-inspur2.ops.haodf.bj1            1/1       Running   1          16h       10.1.101.202   inspur2.ops.haodf.bj1
```
*以上信息有这个 inspur3.ops.haodf.bj1从节点的信息，flannel和proxy都有两个pod*
```
[root@inspur2 ~]# kubectl get nodes
NAME                    STATUS    ROLES     AGE       VERSION
inspur2.ops.bds.bj1   Ready     master    16h       v1.11.3
inspur3.ops.bds.bj1   Ready     <none>    4m        v1.11.3
```
### 忘记初始master节点时的node节点加入集群命令
```
kubeadm token create $token --print-join-command --ttl=0
```
*k8s通过kubeadm搭建集群成功*
