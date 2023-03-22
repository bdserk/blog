## 安装kernel-devel和驱动模块
```
yum install kernel-devel-$(uname -r)  -y
yum remove uacce hisi_zip   hisi_sec2 hisi_hpre hisi_rde -y
yum install uacce hisi_zip   hisi_sec2 hisi_hpre hisi_rde -y
```
## 使用rpm -qa | grep查看升级后的软件版本
```
rpm -qa uacce hisi_zip
```
## 加载驱动
```
modprobe hisi_zip && modprobe hisi_qm && modprobe hisi_sec2 && modprobe hisi_hpre && modprobe hisi_zip  && modprobe hisi_rde
```
## 安装完License之后，如何验证License有没有正常使能鲲鹏加速引擎硬件设备？
```
lspci |grep "ZIP"
```
## 查看模块
```
ls -al /lib/modules/$(uname -r)/extra
```
## 查询内核中已加载的驱动
```
lsmod | grep uacce
```
## 检查动态库链接,包含libwd.so.1字样代表成功
```
ldd /lib64/libz.so.1
```
## 检查虚拟文件系统下是否有相应设备
```
ls -al /sys/class/uacce/
```
## 检查硬件加速器队列
```
cat /sys/class/uacce/hisi_zip-*/attrs/available_instances
yum install flex bison rpcgen 
```
