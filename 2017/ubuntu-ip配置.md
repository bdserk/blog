HP 工作站

访问： ssh xxx@x.x.x.x 密码：123456

修改IP配置 
cat  /etc/network/interface 
auto enp4s0f2
iface enp4s0f2  inet static
   address 10.43.169.x
   netmask 255.255.255.0 
   gateway 10.44.169.x

Dell 工作站
访问： ssh xxxx@x.x.x.x   密码：123456

修改IP 配置
cat  /etc/network/interface 
auto eno1
iface eno1  inet static
   address 10.43.169.x
   netmask 255.255.255.0   
   gateway 10.44.169.x   

重启网卡
/etc/init.d/network restart
