## 双网卡绑定
### 1.确定要做bond的网卡
```
如 ifcfg-em2 和 ifcfg-em3
cp   ifcfg-em2   ifcfg-bond0
```
### 2.配置bond0文件
```shell
vim  ifcfg-bond0
DEVICE=bond0
TYPE=Ethernet
ONBOOT=yes
IPV6INIT=no
USERCTL=no
PEERDNS=yes
BONDING_OPTS="mode=4  miimon=100"
BOOTPROTO=none
IPADDR=
GATEWAY=
NETMASK=
```
### 3.配置em2文件
```shell
vim ifcfg-em2
DEVICE=em2
HWADDR=
TYPE=Ethernet
UUID=
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
IPV6INIT=no
USERCTL=no
MASTER=bond0
SLAVE=yes
```
### 4.配置em3文件
```shell
vim ifcfg-em3
DEVICE=em3
HWADDR=
TYPE=Ethernet
UUID=
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
IPV6INIT=no
USERCTL=no
MASTER=bond0
SLAVE=yes
```
